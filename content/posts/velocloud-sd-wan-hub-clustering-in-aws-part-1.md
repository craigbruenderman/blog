---
draft: false
title: VeloCloud SD-WAN Hub Clustering in AWS (Part 1)
description: 
date: 2024-10-24
image: /images/velocloud.jpg
categories:
    - Networking
    - SD-WAN
tags:
    - AWS
    - SD-WAN
    - Transit Gateway
    - Velocloud
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