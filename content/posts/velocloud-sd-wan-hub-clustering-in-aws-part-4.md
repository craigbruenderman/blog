---
title: VeloCloud SD-WAN Hub Clustering in AWS (Part 4)
description: 
date: 2024-10-27
image: /images/velocloud.png
categories:
    - Networking
    - SD-WAN
tags:
    - AWS
    - SD-WAN
    - Transit Gateway
    - Velocloud
---

# Part 4: Hub Cluster Integration

In Part 3, I laid all the AWS network plumbing groundwork for two Velocloud Hub clusters, one in US-East-1 and the other in US-East-2. So far, not much is happening between the VCE Clusters and AWS and there is no forwarding in place from the SD-WAN overlay to AWS.

## Velocloud Cluster Deployment in AWS

Now I need to do some work in the VCO to configure Hub Profiles and the Edges themselves to get BGP up and running between the Cluster members and TGWs.

## Test Branch Configuration

These Hub Clusters aren’t much use if I don’t have a Branch Edge to tunnel to them. I’ll build a sample Branch at my home office with a workstation behind it for validation. I’ll build a Profile, then spawn an Edge from it.

### Test Branch Profile Configuration

The cobblers kids are the last to get shoes, so we’re kicking it old school with a VCE 510. I’m setting GE3 & GE4 as WAN transport links, and GE1 & GE2 as access ports in good old VLAN 1. I also enabled a simple DHCP server on this SVI.

![](/images/craig-branch-profile.png)

I enable Cloud VPN and add the two Clusters as Hubs.

![](/images/craig-branch-profile-hubs.png)

### Edge Creation and Configuration

I then create and activate an Edge using this Profile, as normal. There’s not much needed for a simple test Branch other than to set the addressing for the VLAN 1 interface.

![](/images/branch-vlan1-settings.png)

## CLI Access

Since I haven’t mentioned it before, make sure to enable Support Access and specify your source address to enable SSH directly to an Edge LAN or WAN transport interfaces. Also make sure tcp/22 is permitted in any Security Groups you have in the case of a virtual Edge.

For hardware edges, the username is either root or vcadmin and the password is VeloHello[last 3 digits of S/N]. Shhh. For virtual Edges, you’ll either need to use a keypair, or have set the password with cloud-init.

![](/images/profile-support-access.png)

Craig-Test-Branch WAN interface sits behind a PAT device, thus the RFC1918 permit

## VCO Configuration of TGW BGP Peering

In the VMware Orchestrator, go to Network Services > Non SD-WAN Destinations via Edge and configure Non SD-WAN Destinations (NSDs) via Edge to the AWS TGWs. This is the first component towards GRE Tunnels.

![](/images/nsd-via-edge-us-east1.png)

![](/images/nsd-via-edge-us-east2.png)

**Note:** Although AWS TGW is able to form multiple Connect Attachments to bond up to 4 x 5Gbps connections, Velocloud documentation makes it clear this won’t help due to lack of ECMP support. “The Edge does not support ECMP across multiple tunnels. Therefore, only one GRE Tunnel will be used for egress Traffic.”

## Cluster Configuration

### Cluster Profile Configuration

I have already created two Profiles, one representing each Cluster. I will use the same Profile for both members of each Cluster, respectively. I need to configure Non SD-WAN Destination via Edge (NSD via Edge) in the VCO Profiles.

![](/images/us-east1-cluster-nsd.png)

NSD via Edge for us-east1 Cluster Profile

![](/images/us-east2-cluster-nsd.png)

NSD via Edge for us-east2 Cluster Profile

### Cluster Member Edge Configuration

Now I need to go through each of the four Edges and add TGW Peers. For the specific NSD, configure the GRE tunnel parameters by selecting the + sign. Configure the following below:
* Tunnel Source is the LAN (GE3) interface
* Tunnel Source IP as the IP address configured on the LAN interface, if specified dynamically use Remote Diagnostics > Interface Stats to obtain the IP address
* TGW ASN
* The Primary Tunnel parameters can be configured by providing, Destination IP, the IP address provided on the TGW Connect Peer
* The Internal Network/Mask must be the same as specified in the TGW Connect Peer Inside configuration

#### us-e1a-vce1

![](/images/us-e1a-vce1-tgw-connect.png)

The BGP knobs and neighbors get configured automatically from the above. I’ll want to apply some filtering later too.

![](/images/us-e1a-vce1-bgp-neighbors.png)

#### us-e1b-vce2

![](/images/us-e1b-vce2-tgw-connect.png)

#### us-e2a-vce1

![](/images/us-e2a-vce1-tgw-connect.png)

#### us-e2b-vce2

![](/images/us-e2b-vce2-tgw-connect.png)

### Static Route on Edges

Velocloud’s documentation stated to configure a static route on each Edge as well. I didn’t find this necessary and my deployment works fine, so I’m leaving it off for now.

## TGW / Cluster Peering Validation

## AWS Validation

![](/images/tgw-peers-up.png)

TGW view of peers

![](/images/branch-prefixes-in-rt.png)

My test Edge prefix 192.168.111.0/24 being learned

## VCO Validation

VCO offers the best global view of BGP peer health, since you can find them all in one place.

### Check NSD via Edge Peers

![](/images/vco-tgw-peers.png)

The 8 peers I expect

### Check if Test Branch is tunneling to Hub Clusters

![](/images/branch-paths-to-clusters.png)

Paths to both Clusters

I’m also looking at the OFC for confirmation that my AWS VPC prefixes are being learned.

![](/images/vco-aws-prefixes.png)

Prefixes inbound from AWS eBGP peers

![](/images/chefs-kiss.jpg)

## CLI Validation

I like to overdo things, so I’ll to SSH into Edges for validation steps and tcpdumping. A debugging script called debug.py lives on them.

{{< highlight commands >}}
debug.py --help
debug.py --bgpd_dump
{{< /highlight >}}

I prefer using the vtysh frontend to the FRRouting engine, and issuing familiar commands directly.

![](/images/vce-bgp-table.png)

Output from one of the us-east2 Cluster members

## Drum Roll Please

I won’t go through every Edge and view on this in the blog, but definitely would in production. At this point, I’m pretty sure this is going to work. Let’s check on the test Linux workloads I built in each Region compute VPC.

![](/images/trace-to-compute-vpc.png)

![](/images/ping-test-compute-vpc-workload.png)

2/2, not bad.

## Part 4 Wrap Up

That was quite a bit of work already, and it definitely did not work the first try. Join me in Part 5 where I do some failure mode testing to ensure I understand various failure scenario behavior and have achieved the degree of HA I’m looking for.

{{< youtube 39YdC-y6QIw >}}