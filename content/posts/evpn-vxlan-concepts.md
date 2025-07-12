---
title: EVPN VXLAN Concepts
draft: true
date: 2025-07-04
image: /images/evpn_vxlan.png
categories:
  - Networking
  - EVPN
tags:
  - EVPN
  - VXLAN
---

## Overview

Enterprise networks

## Underlay Networks

## Overlay Networks

## VXLAN

VXLAN is an IETF RFC 7348 defined data-plane encapsulation protocol for transporting Layer-2 inside Layer-3 using IP/UDP. This approach takes a Layer-2 frame, adds a new VXLAN header to it while mapping the Layer-2 VLAN to a new construct known as a VXLAN Network Identifiers (VNIs), and encapsulates that newly formatted frame in IP/UDP before transporting amongst gateway devices functioning as VXLAN Tunnel Endpoints (VTEPs).

With encapsulation comes additional headers, which means we have to pay attention to MTU. The VXLAN header is 8-bytes, with 24-bits of that being allocated for the VNI. VXLAN adds up to 54-bytes to a standard Ethernet frame, and VTEPs are not allowed to fragment packets, so we need a minimum MTU of 1554-bytes in the transport path between VTEPs.

![](/images/vxlan-header-diag.png)

### VXLAN Tunnel Endpoint (VTEP)

VTEPs are simply multilayer switches with added hardware capabilities needed to perform this encap/decap, and some other supporting bits and bobs. The chipsets in these VTEPs evolved a bit over time, so not all multilayer switches support VXLAN at all, while others support only subsets of currently used VXLAN techniques.

### VXLAN Network Identifier (VNI)

The VXLAN header has a component known as the VNI, which is functionally similar to the 12-bit VLAN ID (VID) field of an 802.1q header in the Layer-2 world. The VNI is a 24-bit value which serves as the identifier of each broadcast domain VXLAN encapsulates. The 24-bits provide up to 16 million unique VNIs, versus the 4096 VIDs in 802.1q.

As we’ll see, VNIs come in L2VNI and L3VNI flavors, depending on their function. For now, focus on L2VNI, and think of it as a mapping that goes into this new VXLAN frame to keep track of which VLAN to map to.

### MP-BGP

* Address Family Identifier
* Subsequent Address Family Identifier
* AFI 25 = L2VPN
* SAFI 70 = EVPN
  * Indicates the updates contain MAC addresses
* Ethernet VPN is used as an improved control plane for VXLAN
* EVPN is an address family found within MP-BGP
* BGPv4 can only carry IPv4 NLRI
* MP-BGP can carry multiple address families
* EVPN is also used as a control plane for other data planes besides VXLAN
  * E.g., MPLS, PBB, NVO
* MP-BGP UPDATE messages are different
  * Type 14 update MP_REACH_MLRI adds routes
  * Type 15 update MP_UNREACH_NLRI delete routes

### Route Distinguisher

* Locally significant value that is configured on each router
* Used in order to uniquely identify a prefix in the global routing table
* 1:1 association with a VRF
* Can be written
  * ASN:nn
  * IP Address:nn

### Route Targets

* Identifies the VRF a prefix belongs to on the receiving router
* Globally significant among all routers
* 1:1 association with a VRF

## EVPN VXLAN Forwarding

### L2EVPN

In L2EVPN, the whole fabric acts like a single large Layer-2 switch.

### L3EVPN

In L3VPN, the fabric adds in the ability to perform inter-VLAN and general routing.

### BUM Traffic Handling

#### Head End Replication (HER)

* To deal with flooding, one option is to manually define a flood list on each Leaf
* The flood list is a map of which VTEPs have which VLANs
* The Leaf then sends encapsulates and sends multiple unicast messages, one to each VTEP in the list
* When receiving VTEP gets the flood, it learns the source MAC into its MAC-VRF table
* Downside is it is manual and ineffcient

### Inclusive Multicast Ethernet Tag

## EVPN Active-Active Multi-Homing
