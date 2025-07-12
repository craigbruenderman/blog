---
title: "NSX-T Notes"
author: Craig Bruenderman
geometry: margin=2cm
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["NSX-T"]
---

# NSX-T Management and Control Plane

## NSX-T vs NSX-V

* NSX-V is managed using vCenter-based tools
* NSX-T management is decoupled from vCenter
* NSX-T supports ESXi, KVM, bare-metal, Kubernetes, OpenShift, AWS, and Azure
* Networking solution for AWS Outposts
* NSX-T 2.4 has feature parity with NSX-V

### NSX-T Features

* Supports VMs and containers
* Every node hosts a management plane agent

### NSX-V vs NSX-T Installation

* NSX-V has NSX Manager registered with vCenter
* NSX-T is a stand-alone solution, but can integrate to vCenter in order to register hosts
* NSX-V has separate appliances for NSX Manager and the NSX Controller cluster
* NSX-T Manager 2.4 include both in the same virtual appliance
  * CLuster of 3
* NSX-V uses VXLAN as its underlay
* NSX-T uses Geneve as its underlay

### NSX-T Licensing

* You can use existing NSX-V licensing
* License is same for T or V

### NSX-T N-VDS Switch

* NSX-V Logical Switches use the vSphere Distributed Switch
* N-VDS is decoupled from VMWare vCenter
  * Host switch that is created on transport nodes
  * Cross-platform support for KVM, etc

## Management, Control and Data Planes

### Management Plane

* NSX Manager Cluster is 3-Node cluster of virtual appliances
* Provides the UI
* Allows integration with cloud management platforms
* Hosts an API
  
### Control Plane

* Runs on same NSX Manager Cluster, unlike in NSX-V where this was separate
* Supervisory function for:
  * Logical Switching
  * Logical Routing
  * Distributed Firewall
  
### Policy Manager

* Runs on same NSX Manager Cluster
* GUI or API
* Push intent into Policy Manager and it implements

### Data Plane (Private Cloud)

* Running on endpoint hosts which run your VMs and containers (ESXi, KVM, etc)
* NSX Edge is part of the data plane
  * North/South router boundary between NSX domain and rest of network
  * NAT, FW, LB, etc

### Data Plane (Public Cloud)

* AWS
* VMC on AWS
* Azure
* IBM

## NSX Manager Architecture

### NSX-T Reference Design

* See the guide

### NSX Manager

* 3-node cluster of VM's (ESXi or KVM)
* 3 Roles
  * Policy role
  * Manager role
  * Controller role
* Need at least 2 of 3 running at all times
  * Don't forget DRS anti-affinity rule
  * Put them on different Datastores
* Distributed replicated database

### Virtual IP (VIP)

* Normally, all nodes must be on same subnet
* Each node has a unique MAC address
* Election is held, similar to VRRP
* Elected Leader owns the VIP
* ARP table will associate VIP with Leader node MAC
* Upon failure, election happens and new Leader sends GARP
* This mechanism provide high-availability only
* Each node also has a unique node IP for IPC

### Load Balancing

* Can also use an external load balancer to run the VIP
* That allows NSX Manager nodes to live in different subnets
* All nodes are active, so can hit any of the 3 for scaling
* Do you want to deal with this extra complexity?

## NSX Controller Concepts

### NSX Manager Cluster

* NSX Manager and Controller no longer separate in NSX-T
* NSX Controller runs on the Manager Cluster

#### NSX Controller Functions

* Logical Switching
  * Tracks MACs of VMs and which Transport Nodes they live on
  * When a VM powers on, it's up to the Transport Node to register the MAC with the NSX Controller and other Transport Nodes Local Control Plane (LCP)
* Central Control Plane (CCP) runs on NSX Controller
* Logical Routing
  * Learns and dsitrbutes routing protocol updates
* Distributed Firewall

## NSX Controller Plane Sharding

* Each Transport Node is now controlled by one of the 3 Controllers
* The TN's are divided among available Controllers

# Preparing Transport Nodes for NSX-T

## NSX-T Data Plane

### Data Plane and Transport Nodes

* Data Plane
* Hypervisors
* Bare metal servers
* NSX Edge
* Transport Zone
* N-VDS
* Tunnel Endpoints (TEPs)
* VMWare Installation Bundles (VIBs)

### ESXi Host VIBs

* N-VDS running on ESXi Host
* LCP
* Management Plane Agent (MPA)

### The Big Picture

* VNI identified a Layer 2 segment
* Distributed Router is the construct that routes between VNIs
* Distributed Firewall allows firewall rules directly applied to VM interface
* Edge Node north/south boundary that connects to external physical network and can provide services (VPN, LB, etc)

## GENEVE and TEPs

### What is a TEP?

* VMKernel Port
* Encapsulates and decapsulates traffic
* Adds headers when sending
* Removes headers when receiving
* GENEVE is new overlay network replacing VXLAN, which was used in NSX-V

## Transport Zones

### Transport Zones

* A colelction of transport nodes that are connected by GENEVE overlay
* They can be ESXi, KVM, Bare Metal, or NSX Edge
* Overlay Transport Zone

# Design

## Deployment Models

* Three deployment models
  * Distributed security for virtual workloads
  * Network virtualization with distributed security for virtual workloads
  * Bare metal workload security via gateway firewall (centralized security) or NSX bare metal agent

### Distributed security model

* 

### Networking and security model

* 

### Centralized security

## Physical Network Requirements


### For Network and Security deployment model
* IP connectivty between all components of NSX and the compute hosts
  * ESXi host management interfaces
  * Edge node VMs or bare metal servers
* IP and MAC mobility across switches within a rack
* MTU >= 1700, 9000 greatly desired
* VM MTU should be at least 200 bytes lower than physical MTU
  * E.g., 1500b MTU on VM with 1700b physical MTU

## Topologies

### Layer 3 leaf/spine with L2 adjacent TORs

* Leaf to spine via routed P2P links
* Leaves deployed in pairs in a rack, sharing common L2 domain
* Compute racks typically require 4 VLANs
  * ESXi Management
  * vMotion
  * IP Storage
  * NSX Overlay

## Manager Cluster

* At least four vSphere Hosts
  * This is to adhere to best practices around vSphere HA and vSphere Dynamic Resource Scheduling (DRS) and allow all three NSX Manager Nodes to remain available during proactive maintenance or a failure scenario
* The hosts should all have access to the same data stores hosting the NSX Manager Nodes to enable DRS and vSphere HA
* Each NSX Manager Node should be deployed onto a different data store (this is supported as a VMFS, NFS, or other data store technology supported by vSphere)
* DRS Anti-Affinity rules should be put in place to prevent, whenever possible, two NSX Manager VMs from running on the same host
* During lifecycle events of this cluster, each node should be independently put into maintenance mode, moving any running NSX Manager Node off the host prior to maintenance, or any NSX Manager Node should be manually moved to a different host
* NSX Manager backups should be configured and pointed to a location running outside of the vSphere Cluster that the NSX Manager Nodes are deployed on
* 