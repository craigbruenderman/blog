---
title: VeloCloud SD-WAN Hub Clustering in AWS (Part 3)
description: 
date: "2024-10-26"
series: ["VeloCloud SD-WAN Hub Clustering in AWS"]
series_order: 3
categories: ["Networking"]
tags: ["AWS", "SD-WAN", "Transit Gateway", "Velocloud"]
---

# Part 3: AWS Network Plumbing

Part 3 picks up at deploying and configuring various AWS network constructs. There are bits of information that will be needed from the Velocloud edge (VCE) deployments in Part 2. Some of the difficulty with this build is just keeping track of the many objects and properties involved.

## Create Transit Gateways (TGWs)

As covered in Part 1, the scope of TGW is per-VPC. I have two Regional network VPCs, so I’ll need to instantiate a TGW in the same region as each respective VPC. Since ultimately I will be running BGP with these TGWs, I choose a 2-byte private ASN for each (both 2-byte and 4-byte are supported). These ASNs will be used later in the BGP configuration on the VMware SD-WAN Edge.

I need to choose TGW CIDR blocks /24 or larger, and it should be outside of any CIDRs assigned to any existing VPCs or other AWS network constructs. /29’s will later be cut from this block and used as GRE endpoints on the AWS TGW when establishing Transit Gateway Connect peers for a TGW attachment. Choose any public or private IP address range, except for addresses in the 169.254.[0-5].0/16 range, and ranges that overlap with addresses for your VPC attachments and on-premises networks.

### us-east-1 TGW

![](/images/create-us-e1-tgw.png)

Creating us-east-1 TGW

![](/images/us-east1-tgw.png)

### us-east-2 TGW

![](/images/create-us-e2-tgw.png)

Creating us-east-2 TGW

![](/images/us-east2-tgw.png)

## Create TGW VPC Attachments to net VPC

To associate each TGW to the net VPCs containing VCEs in its Region, I create TGW Attachments of type VPC. This is the first step towards stitching VCEs to TGWs. Choose the subnets where the VCE LAN side interfaces reside. Since I have VCEs in two AZs, I choose those two private subnets.

### us-east1-tgw Attachment

![](/images/us-east1-tgw-attach-net-vpc.png)

us-east-1 TGW VPC attachment to us-east-1 net VPC

### us-east2-tgw Attachment

![](/images/us-east2-tgw-attach-net-vpc.png)

us-east-2 TGW VPC attachment to us-east-2 net VPC

## Create TGW Connect Attachments

Next I create a TGW attachment of type Connect. This sets up a channel that will provide the GRE tunnel that we’ll use to form the BGP peer between the Edge and TGW. Connect Attachments do not support static routes, only BGP, which is fine since that’s what I want anyway. Connect Attachments also have a 5Gbps limit, and you can theoretically* form up to 4 of them if you need more throughput.

The addressing used later for this BGP peering will come from 169.254.0.0/16, but AWS reserves 169.254.0.0 – 169.254.5.255, so we’ll pick something above that. Layers o’plenty.

![](/images/tgw-connect-attach-logical.png)

TGW Connect conceptual view

### us-east1-tgw Connect Attachment

Notice how this Connect Attachment references the existing TGW VPC attachment as a dependency.

![](/images/us-east1-tgw-attach-connect.png)

**Note** this attachment name got clipped in my screenshot and is actually us-east1-tgw-attach-connect.

Notice the Connect protocol is GRE.

![](/images/us-east1-connect-attached.png)

Connect attachment created

### us-east2-tgw Connect Attachment

![](/images/us-east2-tgw-attach-connect.png)

![](/images/us-east2-connect-attached.png)

Connect attachment created

## Create TGW Connect Peers

With the TGW Connect attachments in place, next we need to create Connect Peers. This sets up the BGP peering connection between TGW and our VCEs. There will be two Connect Attachments per TGW, since each TGW is going to peer with a two nodes of a Velocloud Hub Cluster.

The **Peer IP address** (GRE outer IP address) is the address on the VCE LAN side (GE3) of the Connect peer. This address was DHCP assigned to each VCEs GE3 interface, and can be found either by lookin up the ENI details, or checking Interface Status in the VCO.

The **Transit gateway address** (GRE outer IP address) is the address on the transit gateway side of the Connect peer. The IP address must be specified from the transit gateway CIDR block, and must be unique across Connect attachments on the TGW. If you don’t specify an IP address, AWS uses the first available address from the transit gateway CIDR block. This address can be IPv4 or IPv6, but it must be the same IP address family as the peer IP address.

You can add a transit gateway CIDR block when you create or modify a transit gateway. I already added it upon creation above, so those values are known (172.16.40.0/24 and 172.168.50.0/24).

### us-east1-tgw Connect Peer for us-e1a-vce1

* 10.50.1.50 is the GRE Outside IP on us-e1a-vce1 GE3
* 169.254.6.0/29 is the Inside CIDR block I chose, which is used for the BGP neighbors
    * 169.254.6.1 for BGP on the VCE
    * 169.254.6.2 and 169.254.6.3 for BGP on the us-east1 TGW
* 64902 is the BGP ASN configured on the us-east1 TGW
* 64912 is the BGP ASN configured on us-e1a-vce1

![](/images/us-east1-tgw-connect-peer.png)

In my experience, this is one of the slower operations for typically snappy AWS operations. Once created, the Connect Peer status is found underneath the TGW Connect attachment menu.

I see that AWS selected 172.16.50.18 from the TGW CIDR 172.16.50.0/24 that was allocated when I created it, so this will be specified in the VCO as the Destination IP for us-e1a-vce1 to TGW peering.

### us-east1-tgw Connect Peer for us-e1b-vce2

* 10.50.1.215 is the GRE Outside IP on us-e1b-vce2 GE3
* 169.254.6.8/29 is the Inside CIDR block I chose, which is used for the BGP neighbors
    * 169.254.6.9 for BGP on the VCE
    * 169.254.6.10 and 169.254.6.11 for BGP on the us-east1 TGW
* 64902 is the BGP ASN configured on the us-east1 TGW
* 64912 is the BGP ASN configured on us-e1b-vce2

![](/images/us-east2-tgw-connect-peer.png)

I see that AWS selected 172.16.50.8 from the TGW CIDR 172.16.50.0/24 that was allocated when I created it, so this will be specified in the VCO as the Destination IP for us-e1b-vce2 to TGW peering.

### us-east2-tgw Connect Peer for us-e1b-vce2

* 10.40.2.109 is the GRE Outside IP address on us-e2b-vce2 GE3
* 169.254.7.8/29 is the Inside CIDR Block I chose, which is used for the BGP neighbors
    * 169.254.7.9 for BGP on the VCE
    * 169.254.7.10 and 169.254.7.11 for BGP on the us-east1 TGW
* 64901 is the BGP ASN configured on the us-east1 TGW
* 64911 is the BGP ASN configured on us-e2b-vce2

### Connect Peers Created

us-east1

![](/images/us-east1-tgw-connect-peers.png)

us-east2

![](/images/us-east2-tgw-connect-peers.png)

## TGW Routing Configuration

Next, I’ll go into the each VPC, and add a route for the TGW CIDR to the route table used by the VCEs LAN private subnets. 

### us-east1-net-vpc

![](/images/us-east1-vpc-net-view.png)

Picking the correct route table us-e1-private-rt

![](/images/us-east1-tgw-routes.png)

Adding the us-east1 TGW CIDR

![](/images/us-e2-private-rt-routes.png)

Route towards TGW /29s is in place

![](/images/us-e1-private-rt-assn.png)

Double check subnet associations are correct

Looks good, and I did same for use-east-2 TGW. At this point, the groundwork is laid for BGP peering between the TGWs and VCEs for both of my AWS Regions. No BGP peering is established yet since the VCE side isn’t configured yet.

## Compute VPC TGW Attachments

We’re not done yet, but I want to go ahead and create the AWS plumbing so that there are workloads to reach once Velocloud SD-WAN is exchanging routes with AWS. I have compute VPCs in each Region where I’ll put test workloads, so I’ll create TGW Attachments of type VPC for each of them. I’ll also default route these test workload Subnets back to its Regional TGW.

### US-East-1

![](/images/us-east1-tgw-attach-compute-vpc.png)

TGW VPC Attachment complete

![](/images/us-e1a-sn-compute-view.png)

Default Route added

### US-East-2

![](/images/us-east2-tgw-attach-compute-vpc.png)

TGW VPC Attachment complete

![](/images/us-e2a-sn-compute-view.png)

Default Route added

## Part 3 Wrap Up

That’s it for the AWS network plumbing. Join me in Part 4 where we fully configure the Velocloud  Hub Clusters and actually forward some packets.