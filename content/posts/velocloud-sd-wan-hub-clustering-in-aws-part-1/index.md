---
title: "VeloCloud SD-WAN Hub Clustering in AWS (Part 1)"
description: 
date: "2024-10-24"
series: ["VeloCloud SD-WAN Hub Clustering in AWS"]
series_order: 1
categories: ["Networking"]
tags: ["AWS", "SD-WAN", "Transit Gateway", "Velocloud"]
---

## Overview

Greetings, this series will cover the design/build of VeloCloud Clusters in a multi-region AWS deployment. This exercise dips into a bunch of different technology skill areas, and I couldn't find anything comprehensive written, so here we are. This will be an ambitious multi-part post, so strap in.

I'm decent with routing/switching & Velocloud, and moderately experienced with AWS, so please comment if I need to correct anything.

## Requirements

There are several approaches to this, depending on how sophisticated your AWS network plumbing is and needs to be. In previous deployments with few VPCs, I took a different approach. This scenario has the following requirements:

* Accommodate 2 or more AWS Regions
* AWS Regions will consist of many VPCs, including Network, Security, and general purpose VPCs for various workloads
* Co-exist with DirectConnect
* Provide VeloCloud edge (VCE) high availability within each AWS Region
* VeloCloud scale of ~150 branch locations and ~300 tunnels, per AWS Region

I didn’t find a very clear document for exactly what I wanted, but I was able to cobble it together from a few different sources.

## How to Eat the Elephant

Here are the parts I've broken this into. I'll update the links as (if) I complete each part.

### Part 1 – Review the design options

* Review the mechanics of various Velocloud HA options
* Some general review of the relevant parts of AWS and scope
* Review AWS Transit Gateway
* Review selected AWS and cluster layout

### Part 2 – Initial VCE Deploy

* Deploy and and configure Velocloud edges in AWS

### Part 3 – AWS Deployment

* Deploy and configure various AWS network constructs
* TGWs and routing

### Part 4 – Hub Cluster Integration

* Further configuration of Velocloud
* NSD via Edge configuration
* Establish BGP peering with AWS TGWs
* Validation through VCO, CLI, and AWS Dashboard

### Part 5 – Failure mode testing

* Break things in various ways to learn what happens
* Add optimizations to increase the durability of the design

### Part 6 – Automation

* I go back and automate the entire build using IAC tools
* Python, Terraform and possibly Ansible

### Part 7 – Troubleshooting

* If I’m feeling really industrious, I’ll do a section on Troubleshooting

## Design Options

[This](https://knowledge.broadcom.com/external/article/312340/how-to-design-redundancy-when-deploying.html) document shows Clustering within a single Region with AWS Transit Gateway, but the Cluster connectivity style is BGP peering over IPSEC to VPN gateways, which I’d like to avoid given the performance limitation/penalty. To be clear, the peering relationship I’m talking about is between VCE Cluster members and AWS for the purpose of injecting branch prefixes into the AWS routing domain.

![](/images/tgw-ipsec.png)

That’s close to what I want, but not exactly. I found [another option](https://docs.vmware.com/en/VMware-SD-WAN/5.4/VMware-SD-WAN-Administration-Guide/GUID-9B4E7972-DD77-4377-A8B1-AFDCAF0496B9.html) where Velocloud now supports BGP over GRE support on LAN. That way I can skip the IPSEC requirement and just form a BGP peer from the LAN side of each cluster member directly to TGW. This should give 5Gbps throughput from the VCEs to TGW, rather than 1.5Gbps in the IPSEC model.

## High Availability with Velocloud

Velocloud has a [few different flavors](https://docs.vmware.com/en/VMware-SD-WAN/5.4/VMware-SD-WAN-Administration-Guide/GUID-245EDF35-85C1-41CE-936C-E184C946BB5B.html) of High Availability designs to choose from. 

### Velocloud Standard HA

Broadly speaking, the simplest option is Velocloud Standard HA, which is a dual-device active/passive setup, similar to many other devices like Cisco ASA. Standard HA gives you relatively quick detection and recovery from a total device failure using an L2 HA link between devices, which I recommend to do with a direct patch cable rather than any intermediate bridges. For many years, there was one specific copper interface that had to used for this purpose, which brought some misery with media converters or intermediate bridges to connect distant HA edge pairs. These days you can choose between copper and SFP-based interfaces, so there really is no reason not to use the direct patch method.

![](/images/vce-standard-ha.png)

### Clustering

[Clustering](https://docs.vmware.com/en/VMware-SD-WAN/5.4/VMware-SD-WAN-Administration-Guide/GUID-352B7F3E-A906-4B7E-9875-45E2D578EB54.html#GUID-352B7F3E-A906-4B7E-9875-45E2D578EB54) is another option, which does provide a measure of High Availability, but it really more about horizontal scale. Velocloud edges have various tunnel-scale limitations, so you scale vertically with larger models until you approach 6,000 tunnels on a 3810, and then you option is to scale out with Clustering.

![](/images/scale-1-1.png)

Public cloud Virtual Edges have their own scale limitations.

![](/images/vvce-scale.png)

The Clustering mechanism uses three utilization factors to decide which Cluster member to steer a Branch Edge to. They are CPU usage, memory usage, and tunnel capacity.  of There are some details I’m skipping here, but the VCO and VCGs coordinate the assignment of Edges to individual Cluster Members. When sizing the number of Cluster members, be mindful of the math, especially if your tunnel scale is large. For example, 10K tunnels distribute fine between a 2-node 3810 Cluster, with an approximate steady state of ~5K tunnels each. In a failure mode where you lose a member and are down to a Cluster of one, all 10K tunnels will exceed the tunnel scale of that single surviving 3810. In short, build Clusters with at least N+1 members.

![](/images/clustering-node-selection.png)

Clustering distributes branch tunnels amongst all configured Cluster members to achieve higher total tunnel-scale numbers. Whichever Cluster member a given Edge tunnels to will inject that Edge’s prefixes from OFC and then advertise those prefixes to upstream L3 device(s) via a dynamic routing protocol (OSPF or BGP).

It does also provide a measure of HA due to the fact that Branches are distributed, and redistributed among available Cluster members. If you lose a Cluster member, the Branches which were assigned to the failed member will be redistributed among surviving Cluster members. Your chosen routing protocol (please choose BGP) will handle the move of those individual Branch prefixes. Keep this in mind as well if you were planning to use route summarization since you’ll need to maintain the individual branch prefixes for this to work.

![](/images/clustering-bgp-peering.png)

Clustering doesn’t need any L2 connectivity between members and only relies on dynamic routing towards an L3 device. This is an HA option which can function within AWS’ routing paradigm, but I’ll need something to peer with within AWS.

## AWS Basics

### Scope of Things

AWS design is largely about understanding the scope of AWS objects. Recall that it all starts with an AWS Account. The top level network container is a Region, belonging to an account. Underneath each Region are a variable number of Availability Zones. The scope of VPC’s and TGWs is a Region. The scope of IGWs is a VPC. The scope of a Subnet and EC2 instance is an Availability Zone. 

### Transit Gateways

AWS Transit Gateway (TGW) were introduced in 2018 to simplify the spaghetti that can be created when the need for inter-VPC networking grows beyond a few VPCs. Since VPC Peering alone does not support transitive routing, you end up with the (N*N-1) / 2 problem to provide a full mesh. With TGW, you can limit that to N peering attachments, one from each VPC to the TGW. TGWs are simple to implement and support up to 5,000 attachments. The attachments can be more than just VPC – VPC as we’ll see, but for general VPC – VPC communications, I can’t think of any reason not to use TGW over plain VPC Peering.

In the Clustering section, I mention the Cluster Members form a dynamic routing relationship with some generic L3 module to exchange Branch and Hub prefixes. In physical Cluster deployments, this is L3 typically a WAN edge module, a firewall, or a campus core module. In the AWS world, the L3 module we’ll peer the cluster to is TGW. TGW provides an attachment type that creates an endpoint for the Velocloud Cluster Members to establish a BGP neighbor-ship with so that they can inject prefixes into the AWS routing domain. [Here](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-best-design-practices.html) are the AWS best practices for TGW design.

## Basic AWS Design

I’m starting with two AWS Regions in my scenario, US-East-1 and US-East-2, but this pattern could easily be expanded to more. I want network reachability from my Edges into both Regions independently, and I want good availability to both. I decided to implement two 2-member Clusters, one in each Region. I’ll place each Cluster Node/Member in a unique AZ to limit the failure domain. This fulfills the requirement of high availability within each Region, and both AZ’s would have to fail for a Regional Cluster to totally fail. For now, I won’t contemplate a failure scenario where an entire Regional Cluster has failed, where I’d have to ingress the other Region and transit between Regions to reach the failed Region, but that’s solvable in a later post.

This is the diagram of what I’m planning to build in AWS. I’m omitting showing the steps of creating Regions, VPCs, IGWs, AZs, and Subnets to keep this post from getting out of control. If commenters request this, I might come back and detail those steps in a prequel.

![](/images/cluster-conceptual-design.png)

## Part 1 Wrap Up

That’s it for Part 1. Join me in Part 2 for the Velocloud Edge deployment portion of this series.