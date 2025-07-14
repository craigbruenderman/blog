---
draft: true
title: "Cisco SD-WAN (Part 1)"
date: "2025-07-05"
categories: ["Networking"]
tags: ["Cisco", "SD-WAN"]
---

# Part 1: Overview

In this series I'll generally discuss the Cisco SD-WAN solution and its components, and then move on to explore the process of using automation to deploy and configure Catalyst SD-WAN edges. Some edges will be deployed inside of AWS VPCs in order to reach workloads inside of AWS from my home lab. 

<!--more-->

## Some History

Viptela was the company which originally built the solution which eventually was acquired by Cisco. Over time and as a result of Cisco purchasing Viptela, there have been name changes to the components of Cisco's SD-WAN solution. This set of components was originally known by its Viptela names. vManage was the management plane web UI, vSmart was the control plane, essentially acting to reflect SD-WAN routes to/from SD-WAN edges, and vBond served the role of validating edges were allowed to join the SD-WAN fabric. 

## Solution Components

Even though there are multiple components to the solution, it's convenient to be able to distinguish between SD-WAN edges, and everything else. Cisco's statement sums it up:

"Cisco SD-WAN has been rebranded to Cisco Catalyst SD-WAN. As part of this rebranding, the vManage name has been changed to SD-WAN Manager, the vSmart name has been changed to SD-WAN Controller, and the vBond name has been changed to SD-WAN Validator. Together, the vManage, vSmart, and vBond will be referred to as the **SD-WAN control components** in this document."

| Component Name | Function |
| -------------- | -------- |
| SD-WAN Manager | Web UI for central device configuration and monitoring |
| SD-WAN Validator | Assists in the automatic onboarding of the SD-WAN routers into the SD-WAN overlay |
| SD-WAN Controller | Builds and maintains the network topology and makes decisions on where traffic flows |
| SD-WAN Edges | Individual SD-WAN devices responsible for forwarding packets based on decisions from the control plane |

These components can be deployed by customers on their own prem, or Cisco can host them for customers. Deploying on-prem is typically not very convenient, and Cisco rightly steers customers away from this option, except in certain situations such as edges having no Internet connectivity, or certain regulatory hurdles. It's not much fun to deploy this on-prem, largely because of the certificate work involved.

![](/images/cisco-sdwan-solution.png)

Cisco also has multiple variants on the on-prem versus Cisco-hosted control components, which is not the easiest thing to keep track of. [This document](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/knowledge-base/CloudOps/b-cisco-sdwan-cloudops/m-cloudops-overview.html#overlay-types) covers the variants. Cisco uses somewhat confusing vernacular here, sometimes referring to the control components as Overlays and Fabrics. They also call single customer control component deployments Dedicated Fabric and Single-Tenant in the same document. I'll call the gerneral group of SD-WAN Manager, Validator, and Controller the control components.

In my lab scenarios, I'll be working with what Cisco currently calls [Cloud-delivered Catalyst SD-WAN](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/cloud-delivered-cisco-catalyst-sd-wan-getting-started-guide/cloud-delivered-getting-started.html), multi-tenant variant (CDCS MT). That means I'll be using control components which are hosted by Cisco (in their AWS billing account), and I'm one of multiple customers using it, although I can only access my portion of resources. I worked with Cisco to choose AWS, although this CDCS offering is available in Azure as well. As I found out the hard way, there are [limitations](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/cloud-delivered-cisco-catalyst-sd-wan-getting-started-guide/cloud-delivered-getting-started.html#c-additional-considerations) when using the CDCS option. My primary use case right now is developing and testing automation, so I'm going to need access to the SD-WAN Manager API. As of this writing, the 20.12.x software versions deployed in CDCS do not expose the API, so I went round and round with TAC to rebuild me on a CDCS deployment of 20.15.x. In general, this control component setup has shifting caveats and is not easy. Cisco customers should expect their Cisco account team SD-WAN specialists to help them navigate the caveats, jargon, and processes.

### SD-WAN Manager

This is the web interface used to monitor, configure, and maintain Catalyst SD-WAN (physical and virtual) edge devices and their connected links in the underlay and overlay network. It is the system you regularly interfact with for Day 0, Day 1, and Day 2 operations. Initial configuration, device deployments, and telemetry all happen here.

### SD-WAN Validator

This software component performs initial authentication of SD-WAN Edge devices and facilitates SD-WAN Controller, Manager, and WAN Edge connectivity. It also has an important role in enabling the communication between devices that sit behind Network Address Translation (NAT).

### SD-WAN Controller

This software component maintains the centralized control plane of the SD-WAN network, meaning it knows about all of the network prefixes behind each SD-WAN edge. It maintains a TLS session to each SD-WAN Edge and reflects routes and distributes policy information via the Overlay Management Protocol (OMP). It also orchestrates the secure data plane connectivity between the SD-WAN Edges by reflecting crypto key information originating from Edges, rather than using IKE.

### SD-WAN Edges

These are the actual hardware or virtual devices deployed at physical sites or in public/private clouds. Edges  establish tunnels for data plane connectivity among the sites over one or more WAN transports. Edges are responsible for traffic forwarding, security, encryption, quality of service (QoS), dynamic routing protocols, and more. There have been a number of devices in this category over time, including the legacy Viptela 100/1000/2000 line and Viptela vSphere/KVM-based vEdge virtual devices, and more recently Cisco ISR/ASR series, CSR1000v, and Catalyst 8000v. There are software differences between these that you'll need to understand for a particular production design.

## Logical Design

Here's the logical design I'll be working with:

![](/images/sdwan-logical.png)