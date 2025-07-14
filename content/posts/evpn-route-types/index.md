---
title: EVPN Route Types
description:
date: 2025-03-24
categories: ["Networking"]
tags: ["EVPN", "VXLAN"]
---

## Overview

There are many route types defined for EVPN. Not all route types are supported by all vendor implementations. I'll cover the first 5, since they are the most important.

### Type 1: Auto Discover Segment Route

* Advertise the existence of Ethernet segments network-wide
* Required when an edge device is multi-homed
* Help avoid loops and ensure that only required multicast traffic is delivered to the pertinent Ethernet segments
* Carry an attribute called the Ethernet Segment Identifier (ESI) which is used for active/active multi-homing

### Type 2: MAC-IP Route

* Advertise locally learned/provisioned MACs to their specific Ethernet segments
* Accompanied by a route distinguisher
* Optionally, can advertise IPs which are endpoint /32s
* Support unicast and broadcast
* Support virtual machine MAC migration

### Type 3: Inclusive Multicast Ethernet Tag (IMET) Route

* Used to advertise the list of VNIs connected behind a certain VTEP so that other VTEPs can dynamically build the flood lists for BUM traffic
  * Also possible to create flood lists manually, but please don’t
* An IMET route sets up a path for broadcast, unknown unicast, and multicast (BUM) traffic between VTEPs on a per-VLAN, per-EVI basis

### Type 4: Ethernet Segment Route

* An Ethernet segment identifier (ESI) allows a device to be multi-homed to two or more VTEPs in single/active or active/active mode
* Advertises the reachability of ESIs
* VTEP that are connected to the same Ethernet segment discover each other through the ESI

### Type 5: IP Prefix Route

* Extends EVPN’s capabilities to Layer 3 by allowing the advertisement of IP prefixes over the EVPN network
* In the control plane, EVPN Type 5 routes are used to advertise IP prefixes for inter-subnet connectivity