---
title: EVPN VXLAN Fundamentals
author: Craig Bruenderman
geometry: margin=2cm
toc: true
output: pdf_document
---

## VXLAN Overview

* Ethernet in IP/UDP encap port 4789
* VTEP
* NVE
* NVI

## VXLAN EVPN Basics

* Leaf/Spine is ideal for east/west traffic flows
* VXLAN requires a mechanism to know which end hosts are behind which VTEP
    * This allows the VTEP to build the location-identity mapping database
    * Identity - identifies an end host using its IP, MAC, etc
    * Location - identifies the VTEP responsible for encap/decap tunnel traffic for that end host
* VXLAN Flood & Learn suffers from the scalability issue because it requires flooding the traffic to learn about end hosts behind a VTEP
* The use of MP-BGP and Ethernet VPN (EVPN) solves the scalability issue in VXLAN F&L by minimizing flooding
* The BGP L2VPN EVPN address-family allows the host MAC, IP, network, VRF, and VTEP info to be carried over MP-BGP
* The underlay network provides the reachability information to reach the VTEP, while the overlay control protocol (MP-BGP EVPN) distributes end-host info
* When a VTEP learns about a host on its local segment (e.g., ARP, DHCP), BGP EVPN distributes and provides this information to other BGP EVPN speaking VTEPs within the underlay network
* As long as the source VTEP continues to detect a host behind it, no EVPN update message is sent out, so other VTEPs don't need to age out any remote host info
* For BUM traffic forwarding, the sourcing VTEP is required to send multi-destination traffic to mutiple VTEPS, either using IP multicast or ingress/head-end replcation
* MP-BGP EVPN has different route types
    * Cisco's implementation of VXLAN EVPN uses types 2, 3, and 5

### EVPN Route Types

* Type-2 Mac/IP Advertisement: provides end-host reachability info; MAC (mandatory) and host (/32) IP address (optional)
* Type-3 Inclusive Multicast Ethernet Tag: is used to create the distribution list for ingress replication
* Type-5 IP Route: provides IP prefix and length advertisement in EVPN

### VXLAN BGP EVPN RD & RT

* VRF allows overlapping IP addresses with isolated routing domains
* MP-BGP uses the RD to differentiate between routes stored in the BGP tables
    * RD is 8-bytes (64-bits) and composed of 3 fields
    * Assign RD to each VRF and add RD to IP to maintain uniqueness among identical routes in different VRFs
* MP-BGP also uses a Route Target, which is an extended BGP community places on a route to control the import and export of BGP prefixes between VRFs
    * By using RT's, VRF routes are exported from BGP VRF into VPN address-family and vice versa
    * RT is 8-bytes with prefix:suffix notation
* For simplification, Cisco provides automated derivation of RDs and RTs
    * For RD, the format is RID:VRF-ID (RD, 10.0.0.11:3)
    * For RT, the format is ASN:NVI (RT, 65501:50001)

### VXLAN BGP EVPN Components

* IGP underlay routing
    * OSPF, EIGRP, IS-IS
* Multicast Underlay Routing
    * PIM sparse mode
    * PIM Anycast RP or Bidir Phantom RP on spines
* BGP Underlay control plane
    * Spines should be route reflectors
* Advertise MP-BGP EVPN EVPN Routes
    * MAC to L2 NVI to VTEP mapping
    * IP to L3 VNI to VTEP mapping
* VXLAN data plane encapsulation

## VXLAN Packet Structure

* Inner header
    * Applies to original L2 frame
* Outer header
    * VXLAN header
        * VXLAN flags
        * Reserved bits
        * VNI
    * Outer UDP header
        * Source port
        * VXLAN port UDP 4789
        * UDP Length
        * Checksum
    * Outer IP header
        * IP Header Misc Data
        * Protocol 0x11 (UDP)
        * Header checksum
        * VTEP's source IP
        * VTEP's dest IP
    * Outer MAC header
        * Destination MAC (NH address)
        * Source MAC (VTEP address)
        * EtherType (0x0800)
* VXLAN encapsulation added 50 bytes of overhead
    * 20 bytes + 8 bytes + 8 bytes + 14 bytes = 50 bytes
    * Add another 4 bytes if dot1q is used

# VXLAN EVPN Endpoint Learning and Forwarding

## VXLAN EVPN L2 Forwarding

## VXLAN EVPN L3 Forwarding

# VXLAN EVPN Configuration

## VXLAN EVPN Underlay Configuration

## VXLAN EVPN Overlay L2 Bridging Configuration

## VXLAN EVPN Overlay L3 Routing Configuration