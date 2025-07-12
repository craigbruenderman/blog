---
title: VeloCloud SD-WAN Hub Clustering in AWS (Part 2)
description: 
date: 2024-10-25
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

# Part 2: Deploying VCE Hub Clusters in AWS

Part 1 gave a design overview of what I want to build. In Part 2, I’ll bootstrap two Velocloud Hub Clusters. Each Cluster will exist in a different AWS Region and consist of two VCE Cluster Members, one in each Availability Zone. INHO, the Velocloud AWS vVCE deployment documentation is not great. It only covers CloudFormation, which not everyone uses. I’ve done this with Python + Terraform as well, but that’s pretty advanced and it would be nice to have a guide for those who just want to click their way through the Velocloud Orchestrator (VCO) and AWS Dashboard. I’ll do my best to document that…

I’m going to start out with some pre-work in the Velocloud Orchestrator, and then move on to actually spawning the VCEs in AWS EC2.

## Network Interfaces

We need to understand a few things about interfaces as they pertain to AWS, the VCE’s underlying Linux OS, and the settings in VCO. Our understanding of network interface cards gets a bit fuzzy in the virtual world, so we have to be more particular when describing a NIC, since perspective matters. There are three perspectives on a NIC to consider in this case:

* AWS EC2 models network interfaces as Elastic Network Interfaces (ENIs) with labels like eni-02f1b1f5ba62d664a, which can be “attached” to EC2 operating system instances
* These ENIs have firewall ruleset directly attached to them in the for of Security Groups
* The underlying Linux operating system of a VCE (physical or virtual) models network interfaces as eth[xx], depending on the order they have been presented to the PCI bus
* The Velocloud Orchestrator models network interfaces as GE[x]

All three of these refer to the same construct, but from different perspectives. We must understand the linkage/mapping between all three to know what NIC is actually being inspected. For those of you with vSphere experience, you already know that MAC address is the common denominator.

## VCO Provisioning

### Velocloud Profiles

This part is pretty straightforward. I’ll create two Profiles, one which will represent each Cluster. I’ll actually create one, the configure it, and then duplicate it for the second Profile. These Profiles will end up nearly identical, but you’ll see later why I went with two instead of just one.

### Profile Creation

![](/images/new-aws-profile.png)

### Profile Configuration

I pruned all of the models out except for Virtual Edge, since that’s all that applies when deploying vVCEs in AWS. The interface configuration can be tricky here, depending on the order of operations you use for activation. If you try to activate your edges by manually, you’ll be in for some confusing behavior as interfaces change roles once you run activate.py from the VCE CLI via a bastion host. My strong suggestion is to do it exactly as I define here and spare yourself the trouble.

The Virtual Edge Profile will list GE1-8, but only GE1-3 are relevant. Configure them as shown below and leave the rest as they are. Change GE1-2 to Routed, and disable WAN Link on GE3.

![](/images/east1-profile.png)

The only other thing needed for now is Cloud VPN

![](/images/cloud-vpn-on.png)

Clone us-east1 Profile to us-east2 Profile

![](/images/copy-profile.png)

### Velocloud Edge Creation

Now Configure > Edges > + ADD EDGE, and the VCO then provides a one-time Activation Key which I take note of.

![](/images/provision-edge.png)

Since I want to build a Cluster, create a Cluster

![](/images/create-cluster.png)

And assign it to the Edge

![](/images/set-cluster.png)

Add another Edge using the same East1 Profile, and then add two more Edges using the East2 Profile. Create the us-east2-cluster1 for those two Edges and assign them to it, similarly to above. Now there are 4 total Edges based on those two regional Profiles. Here are the four Edges as created in VCO, assigned to their Clusters.

![](/images/cluster-assignment.png)

## EC2 Deployment

Next up, I’ll spawn EC2 instances of all four VCE’s. These instances will be manually activated against the Activation Key’s from the VCO, register to the correct tenant, pull their configuration, pull down software updates and reboot. For that to happen, I have to get things correct with the EC2 instance launch.

From the AWS Dashboard, go to EC2 and Launch instance. Search for Velocloud in the Marketplace and choose the Edge, not the Gateway.

### us-e1a-vce1

![](/images/launch-us-ea1-vce1.png)

I’m slimming down the Instance type since this is a lab, and choosing an Key pair for accessing via SSH.

![](/images/vce1-instance-type.png)

I’m choosing the correct network VPC I previously created, and choosing a public subnet in us-east-1a with an auto-assigned public IP. This will create an ENI, attach it to my EC2 instance, map it to eth0 in the VCE OS, and map to GE1 in the VCO. We’ll see that this is actually a throw away interface which will be used for activation only.

![](/images/vce-net-settings.png)

I’m using the Security Group VCE-WAN-SG that I already had to permit tcp/22, udp/2426, udp/161, and ICMP from anywhere. You’ll need to create this, or use the one suggested by the AMI.

![](/images/VCE-WAN-SG.png)

### us-e2b-vce2

**Note:** I’m omitting the steps for creating us-e1b-vce2 and us-e2a-vce1, but I did them similarly, selecting the correct Public Subnets for each so that they are in 4 different Availability Zones.

I switch over to the US-East-2 Region. Key pairs and Security Groups are Region specific, so I have separate objects for those here in US-EAST2. This is the second Edge.

![](/images/launch-us-e2b-vce2.png)

![](/images/us-e2b-vce2-instance-type.png)

![](/images/us-e2b-vce2-net-settings.png)

## It’s Alive

Great, the instance finished pretty quickly. Each currently only has a single ENI from being launched, and therefore a single eth0 interface. This corresponds to GE1 in VCO. Since I assigned a public IPs to those ENIs, I can SSH in using the Public IPs of each Edges from its EC2 Instance details.

Even though that AMI said the username was root, it’s actually vcadmin. I SSH to the public IP using the correct username and my key, which I add for convenience (or use ssh -i)

![](/images/ssh-all-hubs.png)

## More ENIs and Security Groups

These instances currently only have a single ENI representing eth0 –> GE1. I won’t actually use this interface in steady state, but I do need 2 more ENIs for GE3 and GE3, which I will use.

### Security Group

I’m going to create two additional ENIs for GE2 and GE3 which I’ll then attach to the Edge EC2 Instances, but first I want a Security group I named VCE-LAN-SG for LAN side ENIs, which are GE3 from the VCO perspective. I’ll use something obvious in the name and description since it’s important to map these correctly later.

![](/images/VCE-LAN-SG.png)

### ENIs for GE2 WAN Transport Interfaces

I’m creating 4 of these, one for each Edge, and putting them in the correct Public Subnets. I’m reusing the Security Group for the WAN side that was created initially. These will map to eth1 –> GE2 and be used for the WAN side.

![](/images/eni-1.png)

### ENIs for GE3 LAN Interfaces

I’m creating 4 of these, one for each Edge, and putting them in the correct Private Subnets. I’m using the Security Group for the LAN side that was created above. These will map to eth2 –> GE3 and be used for the LAN side.

![](/images/eni-2.png)

### ENI Attachment

With the Instances still running, I’ll add the two additional interfaces (GE2 and GE3) to all Edges.  I’m working with us-e1a-vce1 in these screenshots, but all of them are similar.

![](/images/attach-eni.png)

![](/images/attach-GE2.png)

![](/images/attach-GE3.png)

After attaching ENIs to all four instances, I review them. Notice the order than the ENIs appear. The Device index becomes the PCI bus insertion order of the ENIs into the Linux OS. Then the NICs from the perspective of the Linux OS map to GE interfaces from the perspective of the VCO. I’m also double-checking that the Private IPv4’s are in the Subnets I intended them to be. This is critical to get correct because those ENIs are in different Subnets, with different Security Groups.

![](/images/us-e1a-vce-enis.png)

| ENI | Device Index | Linux Interface | VCO Interface |
| --- | ------------ | --------------- | ------------- |
| eni-025b630f8442ce852 | Index 0 | eth0 | GE1 in us-e1a-public-sn1 | 
| eni-0d65827d78bd61276 | Index 1 | eth1 | GE2 in us-e1a-public-sn1 |
| eni-0af767b6c58cbbf85 | Index 2 | eth2 | GE3 in us-e1a-private-sn1 |

Notice that although I have three ENIs attached to the Instance, only the the one that was attached at launch time shows up since I have not yet rebooted the Instance for the two I just added to get noticed by the OS.

![](/images/ifconfig.png)

I can confirm that the ENI at Device index 0 is eni-025b630f8442ce852 is eth0 to Linux by comparing the MAC (0e:c6:9b:9e:f8:ab)

![](/images/eni-summary-vce2.png)

I’ll attach the other ENIs as follows:

| Edge | ENI | Device Index | Linux Interface | VCO Interface |
| ---- | --- | ------------ | --------------- | ------------- |
| us-e1b-vce2 | eni-071383b551dfbb2c7 | Index 0 | eth0 | GE1 in us-e1b-public-sn2 |
| us-e1b-vce2 | eni-074df3799ea97ef2c | Index 1 | eth1 | GE2 in us-e1b-public-sn2 |
| us-e1b-vce2 | eni-0395e0ad982402306 | Index 2 | eth2 | GE3 in us-e1b-private-sn2 |
| us-e2a-vce1 | eni-0abccbb264a3ca269 | Index 0 | eth0 | GE1 in us-e2a-public-sn1 |
| us-e2a-vce1 | eni-060c87ef382512b0d | Index 1 | eth1 | GE2 in us-e2a-private-sn1 |
| us-e2a-vce1 | eni-00e5aa92c47472818 | Index 2 | eth2 | GE3 in us-e2a-private-sn1 |
| us-e2b-vce2 | eni-0e76c8fa2392e483b | Index 0 | eth0 | GE1 in us-e2b-public-sn2 |
| us-e2b-vce2 | eni-069c3f8c0ec49dcb6 | Index 1 | eth1 | GE2 in us-e2b-public-sn2 |
| us-e2b-vce2 | eni-05314599b09176e0b | Index 2 | eth2 | GE3 in us-e2b-private-sn2 |

## Elastic IP Assignment

I want the WAN-side (GE2) IPs to be permanent, so I create Elastic IPs.

![](/images/us-e1-eips.png)

![](/images/us-e2-eips.png)

And then associate them to the Edge GE2 interfaces.

![](/images/associate-eip-vce1.png)

I performed the same step for the other three 3 Edges.

## Activating the Edges

Now all 4 Edges will have new public EIPs for their ENIs which map to GE2, but only once they are activated and rebooted. I want to activate them via the CLI with a Python script called activate.py. I’ll SSH into us-e2a-vce1, and run the command with the VCO and activation key as arguments. 

![](/images/activate-edge.png)

The SSH connection closed as the Edge rebooted, but the CLI output looks promising. I’ll pop over to the VCO to see what it thinks.

![](/images/vco-events.png)

That also looks promising. I proceed to activate the other three edges in no particular order.

## Part 2 Wrap Up

At this point, all four Edges are online and the Clusters appear healthy.

![](/images/cluster-map.png)

![](/images/cluster-loads.png)

I’m going to cut off Part 2 here, as that was a lot. Join me in Part 3 to continue the work in AWS on Transit Gateway and BGP peering.