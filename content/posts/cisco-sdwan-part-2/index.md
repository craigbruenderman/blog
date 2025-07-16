---
draft: true
title: "Cisco SD-WAN (Part 2)"
date: "2025-07-14"
series: ["Cisco SD-WAN"]
series_order: 2
categories: ["Networking"]
tags: ["Cisco", "SD-WAN"]
---

## Branch Internet Access Methods

With SD-WAN designs, we see three variations on how branch Internet access works.

* Backhaul Internet-bound traffic from branch to datacenter over SD-WAN, inspect and source NAT at the data center
  * Traffic is tunneled over whatever transport(s) are available to the SD-WAN overlay between branches and data center backhaul locations
  * Benefit is this is the traditional method many customers are used to and have policy and operations built around
* Use SASE approach where traffic is tunneled to an Internet-facing security filtering service
  * Traffic is IPSEC or GRE tunneled to these SASE services, thus will use available Internet transport(s) at branches
  * Essentially a centralized firewall, but in the cloud instead of a data center
  * Benefit is there can be multiple cloud-based filtering locations which help with latency and bandwidth scaling versus  backhauling to a data center
* Direct Internet access from the branch
  * This could be source NATted by branch SD-WAN device, or a local branch firewall
  * Benefits here get thin-sliced
  * Something like Guest traffic might be acceptable to directly source NAT from the branch - cheap and cheerful
  * Corporate traffic could be inspected by local firewall, but results in branch scale firewall sprawl

## Segmentation

When traffic is transmitted across the WAN, a label is inserted after the ESP header to identify the VPN that the user’s traffic belongs to when it reaches the remote destination.

## Application Performance Optimization

### Application Aware Routing

## SD-WAN API

### Authentication

### Read Info