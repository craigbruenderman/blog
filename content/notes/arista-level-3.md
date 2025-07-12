---
title: "Arista ACE Level 3 Notes"
author: Craig Bruenderman
geometry: margin=2cm
date: 2025-01-01
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["Certification", "EVPN", "VXLAN", "Arista"]
---

# Cloudvision Overview

* Cloudvision Exchange (CVX)
* Cloudvision Portal (CVP)
  * Config automation controller, telemetry collector
  * Launched 2015

## Deployment

* On-prem (VM or physical appliance)
* Cloud-hosted
* Single supports up to 500 devices
* Cluster supports up to 2500 devices

## Cloudvision Device Requirements

* Sofware minimum EOS 4.20
* Enable Terminattr
* Enable eAPI

## Provisioning New Devices

## Configlets

# L2 Info

* VLANs configured in a trunk group are pruned off all ports that are not associated with the trunk group


# MLAG

* Peer Link is a normal port-channel
  * Use LACP mode active
* VLAN 4094 is only permitted on the MLAG Peer Link Po, and not on any other ports on the switch
* All VLANs must be permitted between the peers on the peer link for correct operation
* Use SVI 4094 and a /31 for the peer
  * This SVI can be reused throughout

## Example

### Leaf 1

```
switch1(config)# vlan 4094
switch1(config-vlan-4094)# trunk group m1peer
switch1(config)# interface ethernet 1-2
switch1(config-if-et1-2)# channel-group 10 mode active
switch1(config-if-et1-2)# interface port-channel 10
switch1(config-if-po10)# switchport mode trunk
switch1(config-if-po10)# switchport trunk group m1peer

switch1(config)# interface vlan 4094
switch1(config-if-vl4094)# ip address 10.0.0.1/30
switch1(config-if-vl4094)# no autostate
switch1(config)# no spanning-tree vlan-id 4094

switch1(config)# mlag configuration            
switch1(config-mlag)# local-interface vlan 4094          
switch1(config-mlag)# peer-address 172.17.0.2      
switch1(config-mlag)# peer-link port-channel 101            
switch1(config-mlag)# domain-id mlag_01               
switch1(config-mlag)# heartbeat-interval 2500            
switch1(config-mlag)# reload-delay 150
```

### Leaf 2

```
switch2(config)# vlan 4094
switch2(config-vlan-4094)# trunk group m1peer
switch2(config)# interface ethernet 1-2
switch2(config-if-et1-2)# channel-group 10 mode active
switch2(config-if-et1-2)# interface port-channel 10
switch2(config-if-po10)# switchport mode trunk
switch2(config-if-po10)# switchport trunk group m1peer

switch2(config)# interface vlan 4094 
switch2(config-if-vl4094)# ip address 10.0.0.2/30
switch2(config-if-vl4094)# no autostate
switch2(config)# no spanning-tree vlan-id 4094

switch2(config)# mlag configuration       
switch2(config-mlag)# local-interface vlan 4094           
switch2(config-mlag)# peer-address 172.17.0.1           
switch2(config-mlag)# peer-link port-channel 201                 
switch2(config-mlag)# domain-id mlag_01            
switch2(config-mlag)# heartbeat-interval 2500          
switch2(config-mlag)# reload-delay 150  
```

# Leaf-Spine Architecture

## Layer 2 (L2LS)

## Layer 3 (L3LS)

# Underlay

## BGP in Leaf Spine

# Overlay - Data Plane

## VXLAN

### VXLAN Terminology

VTEP
: Leaf devices with VXLAN configuration, responsible for VLAN -> VXLAN encap/decap

VNI (L2)
: VXLAN Network Identifier is a 24-bit tag added to the VXLAN header to identifiy the L2 segment

VNI (L3)
: Contains L2 VNI

### VXLAN Headers

* VXLAN UDP/4789

## Why we Need a Control Plane

### Head End Replication (HER)

* To deal with flooding, one option is to manually define a flood list on each Leaf
  * The flood list is a map of which VTEPs have which VLANs
  * The Leaf then sends encapsulates and sends multiple unicast messages, one to each VTEP in the list
* When receiving VTEP gets the flood, it learns the source MAC into its MAC-VRF table
* Downside is it is manual and ineffcient

### VXLAN Controller Service (VCS)

* Arista-only solution using a virtual machine
* Dynamically learns the remote VTEPs and associated VXLAN-enabled L2 segments for each VTEP
* MAC address distribution for each VNI
* Automatically build flood list

```
! leaf1
management cvx
    no shut
    server host x.x.x.x
    source-interface management 1
    vrf mgmt
```

## VXLAN with MLAG

* To make VXLAN work with MLAG, we configure an identical loopback IP on both MLAG peers
* This prevents destination VTEP from flapping in MAC-VRF table
* Loopback 0 is used as the router ID, so that they're all unique in BGP
* Loopback 1 is used for VXLAN
* Second loopback only technically needed when MLAG in use

# Overlay - Control Plane

## MP-BGP EVPN Address Family

### MP-BGP EVPN Control Plane

* Ethernet VPN is used as an improved control plane for VXLAN
* EVPN is an address family found within MP-BGP
* BGPv4 can only carry IPv4 NLRI
* MP-BGP can carry multiple address families
* EVPN is also used as a control plane for other data planes besides VXLAN
  * E.g., MPLS, PBB, NVO
* MP-BGP UPDATE messages are different
  * Type 14 update MP_REACH_MLRI adds routes
  * Type 15 update MP_UNREACH_NLRI delete routes

### MP-BGP Overview

* Address Family Identifier
* Subsequent Address Family Identifier
* AFI 25 = L2VPN
* SAFI 70 = EVPN
  * Indicates the updates contain MAC addresses

#### Configure MP-BGP in Arista

```
service routing protocols model multi-agent
! enables EVPN, requires a reboot

router bgp 65001
  neighbor spines peer group
  neighbor spines remote-as 65000
	neighbor spines maximum-routes 12000
  neighbor 10.1.1.1 peer-group spines
  neighbor 10.1.2.1 peer-group spines
	maximum-paths 3 ecmp 3
	network 192.168.0.1/32
	network 192.168.1.1/32

  address-family ipv4
    neighbor spines activate

  address-family l2vpn evpn
    neighbor ... activate
```

### Building the Underlay Config

* Build the Underlay
  * P2P addressing
  * Loopback addressing
  * Jumbo frames (MTU)
* Configure eBGP between leafs/spines
  * Configure neighbors
  * Advertise loopback
  * ```maximum-paths n ecmp n```
    * n # of Spines
  * ```send community```

### Building the Overlay Config

* Build the Overlay
  * Configure VLANs
  * Configure VXLAN interface
  * Map each VLAN to a VNI
* Configure eBGP
  * ebgp-multihop since we're peering loopback to loopback

# VRF

* When assigning interfaces to VRFs, set the VRF first, then the addressing, just like IOS

```text
vrf instance customer1
vrf instance customer2
ip routing vrf customer1
ip routing vrf customer2
int e1
    no sw
    vrf customer1
    ip addr 10.0.0.1/24
    no shut
int e2
    no sw
    vrf customer2
    ip addr 10.0.0.1/24
    no shut

show vrf
```

## Route Distinguishers

* Locally significant value that is configured on each router
* Used in order to uniquely identify a prefix in the global routing table
* 1:1 association with a VRF
* Can be written
  * ASN:nn
  * IP Address:nn

## Route Targets

* Identifies the VRF a prefix belongs to on the receiving router
* Globally significant among all routers
* 1:1 association with a VRF

## RD & RT with EVPN

* 1 VLAN = 1 VNI = 1 MAC-VRF = 1 RD

```text
! Leaf 1
int vxlan 1
	source-interface lo1
	vlan 10 vni 10010
	vlan 20 vni 10020

router bgp 65001
	address-family evpn
		vlan 10
			rd 192.168.0.1:10010
			route-target both 10010:10010
		vlan 20
			rd 192.168.0.1:10020
			route-target both 10020:10020
```

```text
! Leaf 4
int vxlan 1
	source-interface lo1
	vlan 10 vni 10010
	vlan 20 vni 10020

router bgp 65001
	address-family evpn
		vlan 10
			rd 192.168.0.4:10010
			route-target both 10010:10010
		vlan 20
			rd 192.168.0.4:10020
			route-target both 10020:10020
```

# EVPN Route Types

* There are 11 route types defined by MP-BGP EVPN
* First 5 are the most important
* Route Type 1: Auto Discover Segment Route - For active/active multi-homing
* Route Type 2: MAC-IP Route
	* Advertisement of locally learned/provisioned MACs
	* Optionally, can advertise IPs which are /32s of the endpoint
* Route Type 3: IMET (Inclusive Multicast Ethernet Tag)
	* Used to advertise the list of VNIs connected behind a certain VTEP
	* As a result, the VTEPs can dynamically build the flood lists
* Route Type 4: 
* Route Type 5: IP Prefixes
	* Used to advertise IP prefixes

# Routing in the Overlay

## Routing to External Networks

# Routing Models in EVPN L3LS

## IRB

## Symmetric IRB

* Each VTEP only needs its local VLANs configured
* Each VTEP only needs SVIs configured for local VLANs
* VTEPs do not need to learn about the MACs for VLANs that are not connected locally

## Asymmetric IRB

* All VTEPs need all SVIs configured on them
* All VTEPs need to learn about all MACs from all VLANs
* Local routing is performed by each leaf, and bridging happens on the dest leaf

# MP-BGP EVPN Active/Active Multihoming

* MLAG Limited to only 2 devices
* MLAG Consumes interfaces for the peer link
* MLAG requires extra shared loopback IP for the pair
* This method allows multi-homing to more than 2 devices, up to the port-channel member limit
* Very simple configuration (4 lines)
* Each VTEP maintains a unique loopback IP
* The port-channel configured in multihoming is known as an Ethernet Segment (ES)
* Ethernet Segment Identifier (ESI) is the ID of the port-channel
* Ethernet Virtual Instance (EVI) = VLAN

### For Po to come up
* Same LACP system-id must be configured across VTEPs
* Same LACP priority
* Same LACP opertion key
	* Same port-channel interface number

```text
int e1
	channel-group 10 mode active
int po10
	sw mode trunk
	lacp system-id 0000.0000.aaaa
	evpn ethernet-segment
		identifier [xx]
		route-target import [yy]
```

# Lab Work

## Configlets

While there are three types of Configlets, you can only create two of them. User-created Configlets are “Static” and “Builder”. The third type is “Generated”. Static configlets are exactly what the name says: static CLI that can be added to a switch or container. Configlet Builders are Python scripts that will output static configlets. When this happens, these outputted configlets are called “Generated” (due to being created dynamically). These Generated configlets are linked to the switch that they were built for. If you remove that switch, the Generated configlet will be removed too.

