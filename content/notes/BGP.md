---
title: BGP Notes
date: 2024-10-24
author: Craig Bruenderman
geometry: margin=2cm
output: pdf_document
image: /images/route-bgp.png
categories:
  - Networking
  - EVPN
tags:
  - EVPN
  - VXLAN
  - Arista
---

# Overview

These are my BGP notes.

<!--more-->

## BGP

* BGP decides on prefix vector attributes, not anything link related like IGP's do
* AFI Address Family Identifiers
* SAFI Sub-Address Family Identifiers


## Autonomous Systems

* Public ASNs 1-64511
* Private 64512-65535
* Now also 4-byte with RFC 4893
  * If you ever see AS 23456, that means somewhere in the line was a 2-byte only BGP speaker
  * Real AS_PATH encoded with optional transitive attributes AS4_AGGREGATOR and AS4_PATH

## BGP Peerings

* Like IGP, BGP must first find neighbors to exchange routing information with
* Unlike IGP
  * BGP does not have own transport
  * BGP has different types of neighbors
  * BGP neighbors not discoverable by default
    * IGPs typically use multicast to discover each other
  * BGP neighbors do not have to be directly connected

## BGP Transport

* Uses TCP/179
* BGP neighbor statement tells processes to
  * Listen for remote address on TCP/179
  * Initiate session to remote address on TCP/179
  * If collision, higher router-id becomes TCP client
* TCP server must agree on client's expected source IP, or will sent RST
* By default, client packets are sourced from outgoing interface in routing table
  * Can modify this with update-source

# BGP Messages

* Open
* Keepalive
* Update
* Notification

# BGP Neighbor States

* Idle
  * Device is not initiating any other states
* Connect
  * Waiting for TCP connection to complete
* Active
  * Trying to initiate TCP connection
* OpenSent
  * Has sent Open, waiting for neighbor to send its Open
* OpenConfirm
  * Waiting for neighbor to send its Keepalive
* Established
  * Fully established peering

# Path Attributes

* Well-knowns
  * All routers must recognize this PA
* Optional
  * Routers may or may not support this PA

## Types
* Well-known mandatory
  * This Update must contain the PA or the neighbor will be torn down with a notification error message
  * The 3 are AS_PATH, NEXT_HOP, ORIGIN
* Well-known discretionary
  * PA doesn't have to be present
  * E.g. Local Preference
* Optional transitive
  * Must pass PA to other neighbors
  * E.g. Community
* Optional non-transitive
  * Doesn't have to pass PA to other neighbors
  * E.g. MED

# The Origin Attribute

* An attempt to record where a prefix came from
* 3 Options
  * IGP - Via network statement, and thus presumably known via an IGP
  * EGP - Via legacy EGP, so completely deprecated
  * Incomplete - BGP doesn't know exactly, so likely redistributed
* Used as a path selection consideration

# The AS_PATH Attribute

* When a prefix is sent via eBGP and leaves an AS, the AS of the sender is prepended
* Origin AS is on far right of list
* When a prefix is sent via iBGP nothing added

# The NEXT_HOP Attribute

* eBGP prefixes have the NEXT_HOP set to the neighbor sending
* iBGP does not change the NEXT_HOP when sending
  * Update-self, or IGP reachability is used to address this

# BGP Weight

* Pseudo-proprietary
* Local value assigned to a prefix and not advertised to others
* Value 0 - 65535, with higher = better
* When prefix is locally generated, it gets 32768
  * Otherwise, default weight is 0

# BGP Best Path Selection

## Cisco Best Path Selection
* Highest weight
* Highest LOCAL_PREF
* Prefer locally originated
* Shortest AS_PATH
* Lowest origin type
* Lowest MED
* Prefer eBGP over iBGP
* Lowest IGP metric to the BGP NEXT_HOP
* Oldest path
* Lowest Router ID source
* Minimum cluster list length
* Lowest neighbor address

## Juniper Best Path Selection

* Highest LOCAL_PREF
* Lowest AIGP
* Shortest AS_PATH
* Lowest origin type
* Lowest MED
* Prefer locally originated
* Prefer eBGP over iBGP
* Lowest IGP metric to the BGP NEXT_HOP
* Active path
* Primary router
* Lowest Router ID source
* Minimum cluster list length
* Lowest neighbor address

# eBGP Peerings

* eBGP checks for TTL = 1
* eBGP checks that neighbors live on same subnet
* Either side can be the initiator from an ephemeral port to TCP 179 destination port
* Neighbors can use IPv6 address for peering, and still share IPv4 NLRI

# iBGP Peerings

* iBGP split horizon rule states that iBGP learned prefixes are not passed on to other iBGP neighbors
* We often use IGP advertised loopbacks to form iBGP neighbors

# eBGP Multihop

* In Cisco, when you enable ebgp-multihop, the disable-connected-check is automatically done in the background

# Using BGP Authentication

* Typically done for eBGP, but perfectly valid for iBGP too

# Misc Neighbor Options

* log-neighbor-changes is typically a default since neighbor state is critical
* Hard coding the router-id is a good idea

# The Cisco Network Command

* Method to advertise a prefix which exists in the RT
* Must specify the mask property for non-classful address
* Good idea to just always use it

# Redistributing NLRI in Cisco BGP

* 