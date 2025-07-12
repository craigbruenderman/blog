---
title: "Arista EVPN Sample Config"
author: Craig Bruenderman
date: 2025-01-01
geometry: margin=2cm
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["EVPN", "VXLAN", "Arista"]
---

## Spine 1

### Routing

```
service routing protocols model multi-agent
ip routing
```

### Interfaces

```
interface Loopback0
   description ROUTER_ID
   no shutdown
   ip address 10.255.0.1/32
!
interface Ethernet1
   description P2P_dc1-leaf1a_Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.0/31
!
interface Ethernet2
   description P2P_dc1-leaf1b_Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.4/31
!
interface Ethernet3
   description P2P_dc1-svc2a_Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.8/31
!
interface Ethernet4
   description P2P_dc1-svc2b_Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.12/31
```

### BGP

```
ip prefix-list PL-LOOPBACKS-EVPN-OVERLAY
   seq 10 permit 10.255.0.0/27 eq 32
!
route-map RM-CONN-2-BGP permit 10
   match ip address prefix-list PL-LOOPBACKS-EVPN-OVERLAY
!
router bfd
   multihop interval 300 min-rx 300 multiplier 3
!
peer-filter LEAF-AS-RANGE
   10 match as-range 65001-65199 result accept
!
router bgp 65100
   router-id 10.255.0.1
   bgp log-neighbor-changes
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   !
   neighbor EVPN-OVERLAY-PEERS peer group
   neighbor EVPN-OVERLAY-PEERS next-hop-unchanged
   neighbor EVPN-OVERLAY-PEERS update-source Loopback0
   neighbor EVPN-OVERLAY-PEERS bfd
   neighbor EVPN-OVERLAY-PEERS ebgp-multihop 3
   neighbor EVPN-OVERLAY-PEERS password 7 Q4fqtbqcZ7oQuKfuWtNGRQ==
   neighbor EVPN-OVERLAY-PEERS send-community
   neighbor EVPN-OVERLAY-PEERS maximum-routes 0
   !
   neighbor IPv4-UNDERLAY-PEERS peer group
   neighbor IPv4-UNDERLAY-PEERS password 7 7x4B4rnJhZB438m9+BrBfQ==
   neighbor IPv4-UNDERLAY-PEERS send-community
   neighbor IPv4-UNDERLAY-PEERS maximum-routes 12000
   bgp listen range 10.0.0.0/8 peer-group IPv4-UNDERLAY-PEERS peer-filter LEAF-AS-RANGE
   !
   neighbor 10.255.0.3 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.3 remote-as 65101
   neighbor 10.255.0.3 description dc1-leaf1a_Loopback0
   neighbor 10.255.0.4 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.4 remote-as 65101
   neighbor 10.255.0.4 description dc1-leaf1b_Loopback0
   neighbor 10.255.0.5 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.5 remote-as 65102
   neighbor 10.255.0.5 description dc1-leaf2a_Loopback0
   neighbor 10.255.0.6 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.6 remote-as 65102
   neighbor 10.255.0.6 description dc1-leaf2b_Loopback0
   !
   redistribute connected route-map RM-CONN-2-BGP
   !
   address-family evpn
      neighbor EVPN-OVERLAY-PEERS activate
   !
   address-family ipv4
      no neighbor EVPN-OVERLAY-PEERS activate
      neighbor IPv4-UNDERLAY-PEERS activate
```

## Spine 2

### Interfaces

```
interface Loopback0
   description ROUTER_ID
   no shutdown
   ip address 10.255.0.2/32
!
interface Ethernet1
   description P2P_dc1-leaf1a_Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.2/31
!
interface Ethernet2
   description P2P_dc1-leaf1b_Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.6/31
!
interface Ethernet3
   description P2P_dc1-svc2a_Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.10/31
!
interface Ethernet4
   description P2P_dc1-svc2b_Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.14/31
```

### BGP

```
ip prefix-list PL-LOOPBACKS-EVPN-OVERLAY
   seq 10 permit 10.255.0.0/27 eq 32
!
route-map RM-CONN-2-BGP permit 10
   match ip address prefix-list PL-LOOPBACKS-EVPN-OVERLAY
!
router bfd
   multihop interval 300 min-rx 300 multiplier 3
!
peer-filter LEAF-AS-RANGE
   10 match as-range 65001-65199 result accept
!
router bgp 65100
   router-id 10.255.0.2
   bgp log-neighbor-changes
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   !
   neighbor EVPN-OVERLAY-PEERS peer group
   neighbor EVPN-OVERLAY-PEERS next-hop-unchanged
   neighbor EVPN-OVERLAY-PEERS update-source Loopback0
   neighbor EVPN-OVERLAY-PEERS bfd
   neighbor EVPN-OVERLAY-PEERS ebgp-multihop 3
   neighbor EVPN-OVERLAY-PEERS password 7 Q4fqtbqcZ7oQuKfuWtNGRQ==
   neighbor EVPN-OVERLAY-PEERS send-community
   neighbor EVPN-OVERLAY-PEERS maximum-routes 0
   !
   neighbor IPv4-UNDERLAY-PEERS peer group
   neighbor IPv4-UNDERLAY-PEERS password 7 7x4B4rnJhZB438m9+BrBfQ==
   neighbor IPv4-UNDERLAY-PEERS send-community
   neighbor IPv4-UNDERLAY-PEERS maximum-routes 12000
   bgp listen range 10.0.0.0/8 peer-group IPv4-UNDERLAY-PEERS peer-filter LEAF-AS-RANGE
   !
   neighbor 10.255.0.3 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.3 remote-as 65101
   neighbor 10.255.0.3 description dc1-leaf1a_Loopback0
   neighbor 10.255.0.4 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.4 remote-as 65101
   neighbor 10.255.0.4 description dc1-leaf1b_Loopback0
   neighbor 10.255.0.5 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.5 remote-as 65102
   neighbor 10.255.0.5 description dc1-leaf2a_Loopback0
   neighbor 10.255.0.6 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.6 remote-as 65102
   neighbor 10.255.0.6 description dc1-leaf2b_Loopback0
   !
   redistribute connected route-map RM-CONN-2-BGP
   !
   address-family evpn
      neighbor EVPN-OVERLAY-PEERS activate
   !
   address-family ipv4
      no neighbor EVPN-OVERLAY-PEERS activate
      neighbor IPv4-UNDERLAY-PEERS activate
```

## Spine 3

```
service routing protocols model multi-agent
ip routing

peer-filter LEAF-AS-RANGE
   10 match as-range 65001-65199 result accept

no router bgp
router bgp 65100
router-id 10.23.23.23
maximum-paths 4 ecmp 4
no bgp default ipv4-unicast
bgp log-neighbor-changes

neighbor Leafs peer group
neighbor Leafs maximum-routes 12000
neighbor BorderleafUnderlay peer group
neighbor BorderleafUnderlay maximum-routes 12000
neighbor 10.10.5.1 peer group BorderleafUnderlay
neighbor BorderleafUnderlay remote-as 65500
neighbor 10.10.5.1 description Borderleaf1
bgp listen range 10.0.0.0/8 peer-group Leafs peer-filter LEAF-AS-RANGE
neighbor BorderleafUnderlay peer group
neighbor BorderleafUnderlay remote-as 65500
neighbor 10.10.5.1 peer group BorderleafUnderlay
neighbor 10.10.5.1 description Borderleaf1
address-family ipv4
      neighbor Leafs activate
      redistribute connected
```

## Spine 4

```
service routing protocols model multi-agent
ip routing

peer-filter LEAF-AS-RANGE
   10 match as-range 65001-65199 result accept

no router bgp
router bgp 65100
router-id 10.24.24.24
maximum-paths 4 ecmp 4
no bgp default ipv4-unicast
bgp log-neighbor-changes

neighbor Leafs peer group
neighbor Leafs maximum-routes 12000
neighbor BorderleafUnderlay peer group
neighbor BorderleafUnderlay maximum-routes 12000
neighbor 10.10.5.1 peer group BorderleafUnderlay
neighbor BorderleafUnderlay remote-as 65500
neighbor 10.10.5.1 description Borderleaf1
bgp listen range 10.0.0.0/8 peer-group Leafs peer-filter LEAF-AS-RANGE
neighbor BorderleafUnderlay peer group
neighbor BorderleafUnderlay remote-as 65500
neighbor 10.10.5.1 peer group BorderleafUnderlay
neighbor 10.10.5.1 description Borderleaf1
address-family ipv4
      neighbor Leafs activate
      redistribute connected

ip prefix-list Loopback
    permit 0.0.0.0 0.0.0.0 eq 32
route-map RedistributeLoopback permit 10
    match ip address prefix-list Loopback
router bgp 65001
address-family ipv4
     redistribute connected route-map RedistributeLoopback

router bgp 65004
   neighbor 10.21.21.21 peer group SpineOVERLAY
   neighbor 10.22.22.22 peer group SpineOVERLAY
   neighbor 10.23.23.23 peer group SpineOVERLAY
   neighbor 10.24.24.24 peer group SpineOVERLAY

   neighbor SpineOVERLAY send-community extended
   neighbor SpineOVERLAY ebgp-multihop 3
   neighbor SpineOVERLAY remote-as 65100
   neighbor SpineOVERLAY update-source loopback 0
   neighbor SpineOVERLAY maximum-routes 0

   neighbor 10.21.21.21 description Spine1OVERLAY
   neighbor 10.22.22.22 description Spine2OVERLAY
   neighbor 10.23.23.23 description Spine3OVERLAY
   neighbor 10.24.24.24 description Spine4OVERLAY

address-family ipv4
   no neighbor SpineOVERLAY activate

 address-family evpn
   neighbor SpineOVERLAY activate

  neighbor BorderleafOVERLAY next-hop-unchanged
  router bgp 65100
   neighbor 10.31.31.31 peer group BorderleafOVERLAY
   neighbor 10.32.32.32 peer group BorderleafOVERLAY

   neighbor 10.11.11.11 remote-as 65001
   neighbor 10.11.11.11 send-community extended
   neighbor 10.11.11.11 ebgp-multihop 3
   neighbor 10.11.11.11 update-source loopback 0
   neighbor 10.11.11.11 maximum-routes 0
   neighbor 10.11.11.11 next-hop-unchanged

   neighbor 10.12.12.12 remote-as 65002
   neighbor 10.12.12.12 send-community extended
   neighbor 10.12.12.12 ebgp-multihop 3
   neighbor 10.12.12.12 update-source loopback 0
   neighbor 10.12.12.12 maximum-routes 0
   neighbor 10.12.12.12 next-hop-unchanged

   neighbor 10.13.13.13 remote-as 65003
   neighbor 10.13.13.13 send-community extended
   neighbor 10.13.13.13 ebgp-multihop 3
   neighbor 10.13.13.13 update-source loopback 0
   neighbor 10.13.13.13 maximum-routes 0
   neighbor 10.13.13.13 next-hop-unchanged

   neighbor 10.14.14.14 remote-as 65004
   neighbor 10.14.14.14 send-community extended
   neighbor 10.14.14.14 ebgp-multihop 3
   neighbor 10.14.14.14 update-source loopback 0
   neighbor 10.14.14.14 maximum-routes 0
   neighbor 10.14.14.14 next-hop-unchanged

   neighbor BorderleafOVERLAY send-community extended
   neighbor BorderleafOVERLAY ebgp-multihop 3
   neighbor BorderleafOVERLAY remote-as 65500
   neighbor BorderleafOVERLAY update-source loopback 0
   neighbor BorderleafOVERLAY maximum-routes 0
   neighbor BorderleafOVERLAY  next-hop-unchanged

   neighbor 10.11.11.11 description Leaf1OVERLAY
   neighbor 10.12.12.12 description Leaf2OVERLAY
   neighbor 10.13.13.13 description Leaf3OVERLAY
   neighbor 10.14.14.14 description Leaf4OVERLAY

   neighbor 10.31.31.31 description Borderleaf1OVERLAY
   neighbor 10.32.32.32 description Borderleaf2OVERLAY

   address-family ipv4
   no neighbor 10.11.11.11 activate
   no neighbor 10.12.12.12 activate
   no neighbor 10.13.13.13 activate
   no neighbor 10.14.14.14 activate
   no neighbor BorderleafOVERLAY activate

   address-family evpn
   neighbor 10.11.11.11 activate
   neighbor 10.12.12.12 activate
   neighbor 10.13.13.13 activate
   neighbor 10.14.14.14 activate
   neighbor BorderleafOVERLAY activate

router bgp 65001
 vlan 101
   rd 10.11.11.11:101
   route-target both 101:10101
   redistribute learned
   no redistribute host-route

router bgp 65002
 vlan 101
   rd 10.12.12.12:101
   route-target both 101:10101
   redistribute learned
   no redistribute host-route

router bgp 65003
 vlan 101
   rd 10.13.13.13:101
   route-target both 101:10101
   redistribute learned
   no redistribute host-route

router bgp 65004
  vlan 101
   rd 10.14.14.14:101
   route-target both 101:10101
   redistribute learned
   no redistribute host-route

interface  vxlan 1
  vxlan source-interface loopback 0
  vxlan udp-port 4789
  vxlan vlan 101 vni 10101


interface e7
  switchport
  switchport mode access
  switchport access vlan 101
  spanning-tree portfast
  no shut

interface  vlan 101
   ip address virtual 192.168.101.254/24
   no autostate

ip virtual-router mac-address 001c.7300.0099

interface e8,9,10
  shutdown

router bgp 65003

vlan 301
   rd 10.14.14.14:3333
   route-target both 3333:3333
   redistribute learned

vlan 401
   rd 10.14.14.14:4444
   route-target both 4444:4444
   redistribute learned

vrf VRF-C
   rd 10.14.14.14:7777
   route-target export evpn 7777:7777
   route-target import evpn 7777:7777
   redistribute connected

vlan 301

interface Port-Channel2
 switchport
 switchport mode access
 switchport access vlan 401
 no shut
 evpn ethernet-segment
   identifier 0050:0c00:0700:0700:bbbb
   route-target import 00:1c:73:6a:b5:3e
   lacp system-id 0000.0000.bbbb
interface Ethernet9
 channel-group 2 mode active
```

## Leaf 1A

### Underlay Interfaces

```
interface Loopback0
   description ROUTER_ID
   no shutdown
   ip address 10.255.0.3/32
!
interface Loopback1
   description VXLAN_TUNNEL_SOURCE
   no shutdown
   ip address 10.255.1.3/32
!
interface Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.1/31
!
interface Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.3/31
```

### MLAG

```
vlan 4093
   name MLAG_L3
   trunk group MLAG
!
vlan 4094
   name MLAG
   trunk group MLAG
!
interface Vlan4093
   description MLAG_L3
   no shutdown
   mtu 9124
   ip address 10.255.1.96/31
!
interface Vlan4094
   description MLAG
   no shutdown
   mtu 9124
   no autostate
   ip address 10.255.1.64/31
!
interface Ethernet3
   no shutdown
   channel-group 3 mode active
!
interface Ethernet4
   no shutdown
   channel-group 3 mode active
!
interface Port-Channel3
   description MLAG_dc1-leaf1b_Port-Channel3
   no shutdown
   switchport mode trunk
   switchport trunk group MLAG
   switchport
!
ip virtual-router mac-address 00:1c:73:00:00:99
mlag configuration
   domain-id DC1_L3_LEAF1
   local-interface Vlan4094
   peer-address 10.255.1.65
   peer-link Port-Channel3
   reload-delay mlag 300
   reload-delay non-mlag 330

## Client Interfaces

int e6
   channel-group 11 mode active
   no shut
int po11
   switchport
   switchport mode access
   switchport access vlan 11
   no shut
   mlag 11
```

### BGP

```
service routing protocols model multi-agent
ip routing
!
ip prefix-list PL-LOOPBACKS-EVPN-OVERLAY
   seq 10 permit 10.255.0.0/27 eq 32
   seq 20 permit 10.255.1.0/27 eq 32
!
ip prefix-list PL-MLAG-PEER-VRFS
   seq 10 permit 10.255.1.96/31
!
route-map RM-CONN-2-BGP permit 10
   match ip address prefix-list PL-LOOPBACKS-EVPN-OVERLAY
!
route-map RM-CONN-2-BGP-VRFS deny 10
   match ip address prefix-list PL-MLAG-PEER-VRFS
!
route-map RM-CONN-2-BGP-VRFS permit 20
!
route-map RM-MLAG-PEER-IN permit 10
   description Make routes learned over MLAG Peer-link less preferred on spines to ensure optimal routing
   set origin incomplete
!
router bgp 65101
   bgp log-neighbor-changes
   router-id 10.255.0.3
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   !
   neighbor IPv4-UNDERLAY-PEERS peer group
   neighbor IPv4-UNDERLAY-PEERS password 7 7x4B4rnJhZB438m9+BrBfQ==
   neighbor IPv4-UNDERLAY-PEERS send-community
   neighbor IPv4-UNDERLAY-PEERS maximum-routes 12000
   !
   neighbor MLAG-IPv4-UNDERLAY-PEER peer group
   neighbor MLAG-IPv4-UNDERLAY-PEER remote-as 65101
   neighbor MLAG-IPv4-UNDERLAY-PEER next-hop-self
   neighbor MLAG-IPv4-UNDERLAY-PEER description dc1-leaf1b
   neighbor MLAG-IPv4-UNDERLAY-PEER route-map RM-MLAG-PEER-IN in
   neighbor MLAG-IPv4-UNDERLAY-PEER password 7 4b21pAdCvWeAqpcKDFMdWw==
   neighbor MLAG-IPv4-UNDERLAY-PEER send-community
   neighbor MLAG-IPv4-UNDERLAY-PEER maximum-routes 12000
   neighbor 10.255.1.97 peer group MLAG-IPv4-UNDERLAY-PEER
   neighbor 10.255.1.97 description MLAG Neighbor
   !
   neighbor 10.255.255.0 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.0 remote-as 65100
   neighbor 10.255.255.0 description dc1-spine1_Ethernet1
   neighbor 10.255.255.2 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.2 remote-as 65100
   neighbor 10.255.255.2 description dc1-spine2_Ethernet1
   redistribute connected route-map RM-CONN-2-BGP
   !
   address-family ipv4
      no neighbor EVPN-OVERLAY-PEERS activate
      neighbor IPv4-UNDERLAY-PEERS activate
      neighbor MLAG-IPv4-UNDERLAY-PEER activate
```

### EVPN

```
router bgp 65101
   neighbor EVPN-OVERLAY-PEERS peer group
   neighbor EVPN-OVERLAY-PEERS update-source Loopback0
   neighbor EVPN-OVERLAY-PEERS bfd
   neighbor EVPN-OVERLAY-PEERS ebgp-multihop 3
   neighbor EVPN-OVERLAY-PEERS password 7 Q4fqtbqcZ7oQuKfuWtNGRQ==
   neighbor EVPN-OVERLAY-PEERS send-community
   neighbor EVPN-OVERLAY-PEERS maximum-routes 0
   neighbor 10.255.0.1 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.1 remote-as 65100
   neighbor 10.255.0.1 description dc1-spine1_Loopback0
   neighbor 10.255.0.2 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.2 remote-as 65100
   neighbor 10.255.0.2 description dc1-spine2_Loopback0
   !
   address-family evpn
      neighbor EVPN-OVERLAY-PEERS activate
   !
   vlan-aware-bundle VRF-10
      rd 10.255.0.3:10
      route-target both 10:10
      redist learned
      vlan 10-49
   !
   vlan-aware-bundle VRF-11
      rd 10.255.0.3:11
      route-target both 11:11
      redist learned
      vlan 50-99
   !
   vrf VRF10
      rd 10.255.0.3:10
      route-target both 10:10
      router-id 10.255.0.3
      neighbor 10.255.1.97 peer group MLAG-IPv4-UNDERLAY-PEER
      redistribute connected route-map RM-CONN-2-BGP-VRFS
   !
   vrf VRF11
      rd 10.255.0.3:11
      route-target both 11:11
      router-id 10.255.0.3
      neighbor 10.255.1.97 peer group MLAG-IPv4-UNDERLAY-PEER
      redistribute connected route-map RM-CONN-2-BGP-VRFS
```

### VXLAN

```
vrf instance VRF10
vrf instance VRF11
ip routing vrf VRF10
ip routing vrf VRF11
!
vlan 10
   name VRF10_VLAN10
!
interface Vlan11
   description VRF10_VLAN10
   no shutdown
   vrf VRF10
   ip address virtual 10.10.10.1/24
!
vlan 11
   name VRF10_VLAN11
!
interface Vlan11
   description VRF10_VLAN11
   no shutdown
   vrf VRF10
   ip address virtual 10.10.11.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan virtual-router encapsulation mac-address mlag-system-id
   vxlan udp-port 4789
   vxlan vlan 10-1000 vni 10010-11000
   vxlan vrf VRF10 vni 10
   vxlan vrf VRF11 vni 11
```

## Leaf 1B

### Underlay Interfaces

```
interface Loopback0
   description ROUTER_ID
   no shutdown
   ip address 10.255.0.4/32
!
interface Loopback1
   description VXLAN_TUNNEL_SOURCE
   no shutdown
   ip address 10.255.1.3/32
!
interface Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.5/31
!
interface Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.7/31
```

### MLAG

```
vlan 4093
   name MLAG_L3
   trunk group MLAG
!
vlan 4094
   name MLAG
   trunk group MLAG
!
interface Vlan4093
   description MLAG_L3
   no shutdown
   mtu 9124
   ip address 10.255.1.97/31
!
interface Vlan4094
   description MLAG
   no shutdown
   mtu 9124
   no autostate
   ip address 10.255.1.65/31
!
interface Ethernet3
   no shutdown
   channel-group 3 mode active
!
interface Ethernet4
   no shutdown
   channel-group 3 mode active
!
interface Port-Channel3
   description MLAG_dc1-leaf2b_Port-Channel3
   no shutdown
   switchport mode trunk
   switchport trunk group MLAG
   switchport
!
ip virtual-router mac-address 00:1c:73:00:00:99
mlag configuration
   domain-id DC1_L3_LEAF1
   local-interface Vlan4094
   peer-address 10.255.1.64
   peer-link Port-Channel3
   reload-delay mlag 300
   reload-delay non-mlag 330
```

### BGP

```
service routing protocols model multi-agent
ip routing
!
ip prefix-list PL-LOOPBACKS-EVPN-OVERLAY
   seq 10 permit 10.255.0.0/27 eq 32
   seq 20 permit 10.255.1.0/27 eq 32
!
ip prefix-list PL-MLAG-PEER-VRFS
   seq 10 permit 10.255.1.96/31
!
route-map RM-CONN-2-BGP permit 10
   match ip address prefix-list PL-LOOPBACKS-EVPN-OVERLAY
!
route-map RM-CONN-2-BGP-VRFS deny 10
   match ip address prefix-list PL-MLAG-PEER-VRFS
!
route-map RM-CONN-2-BGP-VRFS permit 20
!
route-map RM-MLAG-PEER-IN permit 10
   description Make routes learned over MLAG Peer-link less preferred on spines to ensure optimal routing
   set origin incomplete
!
router bgp 65101
   bgp log-neighbor-changes
   router-id 10.255.0.4
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   !
   neighbor IPv4-UNDERLAY-PEERS peer group
   neighbor IPv4-UNDERLAY-PEERS password 7 7x4B4rnJhZB438m9+BrBfQ==
   neighbor IPv4-UNDERLAY-PEERS send-community
   neighbor IPv4-UNDERLAY-PEERS maximum-routes 12000
   !
   neighbor MLAG-IPv4-UNDERLAY-PEER peer group
   neighbor MLAG-IPv4-UNDERLAY-PEER remote-as 65101
   neighbor MLAG-IPv4-UNDERLAY-PEER next-hop-self
   neighbor MLAG-IPv4-UNDERLAY-PEER description dc1-leaf1b
   neighbor MLAG-IPv4-UNDERLAY-PEER route-map RM-MLAG-PEER-IN in
   neighbor MLAG-IPv4-UNDERLAY-PEER password 7 4b21pAdCvWeAqpcKDFMdWw==
   neighbor MLAG-IPv4-UNDERLAY-PEER send-community
   neighbor MLAG-IPv4-UNDERLAY-PEER maximum-routes 12000
   neighbor 10.255.1.96 peer group MLAG-IPv4-UNDERLAY-PEER
   neighbor 10.255.1.96 description MLAG Neighbor
   !
   neighbor 10.255.255.4 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.4 remote-as 65100
   neighbor 10.255.255.4 description dc1-spine1_Ethernet2
   neighbor 10.255.255.6 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.6 remote-as 65100
   neighbor 10.255.255.6 description dc1-spine2_Ethernet2
   redistribute connected route-map RM-CONN-2-BGP
   !
   address-family ipv4
      no neighbor EVPN-OVERLAY-PEERS activate
      neighbor IPv4-UNDERLAY-PEERS activate
      neighbor MLAG-IPv4-UNDERLAY-PEER activate
```

### EVPN

```
router bgp 65101
   neighbor EVPN-OVERLAY-PEERS peer group
   neighbor EVPN-OVERLAY-PEERS update-source Loopback0
   neighbor EVPN-OVERLAY-PEERS bfd
   neighbor EVPN-OVERLAY-PEERS ebgp-multihop 3
   neighbor EVPN-OVERLAY-PEERS password 7 Q4fqtbqcZ7oQuKfuWtNGRQ==
   neighbor EVPN-OVERLAY-PEERS send-community
   neighbor EVPN-OVERLAY-PEERS maximum-routes 0
   neighbor 10.255.0.1 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.1 remote-as 65100
   neighbor 10.255.0.1 description dc1-spine1_Loopback0
   neighbor 10.255.0.2 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.2 remote-as 65100
   neighbor 10.255.0.2 description dc1-spine2_Loopback0
   !
   address-family evpn
      neighbor EVPN-OVERLAY-PEERS activate
   !
   vlan-aware-bundle VRF-10
      rd 10.255.0.4:10
      route-target both 10:10
      redist learned
      vlan 10-49
   !
   vlan-aware-bundle VRF-11
      rd 10.255.0.4:11
      route-target both 11:11
      redist learned
      vlan 50-99
   !
   vrf VRF10
      rd 10.255.0.4:10
      route-target both 10:10
      router-id 10.255.0.4
      neighbor 10.255.1.96 peer group MLAG-IPv4-UNDERLAY-PEER
      neighbor 10.255.1.96 description MLAG Peer
      redistribute connected route-map RM-CONN-2-BGP-VRFS
```

### VXLAN

```
vrf instance VRF10
vrf instance VRF11
ip routing vrf VRF10
ip routing vrf VRF11
!
vlan 10
   name VRF10_VLAN10
!
interface Vlan11
   description VRF10_VLAN10
   no shutdown
   vrf VRF10
   ip address virtual 10.10.10.1/24
!
vlan 11
   name VRF10_VLAN11
!
interface Vlan11
   description VRF10_VLAN11
   no shutdown
   vrf VRF10
   ip address virtual 10.10.11.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan virtual-router encapsulation mac-address mlag-system-id
   vxlan udp-port 4789
   vxlan vlan 10-1000 vni 10010-11000
   vxlan vrf VRF10 vni 10
   vxlan vrf VRF11 vni 11
```

## SVC Leaf A

### Underlay Interfaces

```
interface Loopback0
   description ROUTER_ID
   no shutdown
   ip address 10.255.0.5/32
!
interface Loopback1
   description VXLAN_TUNNEL_SOURCE
   no shutdown
   ip address 10.255.1.5/32
!
interface Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.9/31
!
interface Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.11/31
```

### MLAG

```
vlan 4093
   name MLAG_L3
   trunk group MLAG
!
vlan 4094
   name MLAG
   trunk group MLAG
!
interface Vlan4093
   description MLAG_L3
   no shutdown
   mtu 9124
   ip address 10.255.1.96/31
!
interface Vlan4094
   description MLAG
   no shutdown
   mtu 9124
   no autostate
   ip address 10.255.1.64/31
!
interface Ethernet3
   no shutdown
   channel-group 3 mode active
!
interface Ethernet4
   no shutdown
   channel-group 3 mode active
!
interface Port-Channel3
   description MLAG Po
   no shutdown
   switchport mode trunk
   switchport trunk group MLAG
   switchport
!
ip virtual-router mac-address 00:1c:73:00:00:99
mlag configuration
   domain-id DC1_L3_LEAF2
   local-interface Vlan4094
   peer-address 10.255.1.65
   peer-link Port-Channel3
   reload-delay mlag 300
   reload-delay non-mlag 330
```

### BGP

```
service routing protocols model multi-agent
ip routing
!
ip prefix-list PL-LOOPBACKS-EVPN-OVERLAY
   seq 10 permit 10.255.0.0/27 eq 32
   seq 20 permit 10.255.1.0/27 eq 32
!
ip prefix-list PL-MLAG-PEER-VRFS
   seq 10 permit 10.255.1.96/31
!
route-map RM-CONN-2-BGP permit 10
   match ip address prefix-list PL-LOOPBACKS-EVPN-OVERLAY
!
route-map RM-CONN-2-BGP-VRFS deny 10
   match ip address prefix-list PL-MLAG-PEER-VRFS
!
route-map RM-CONN-2-BGP-VRFS permit 20
!
route-map RM-MLAG-PEER-IN permit 10
   description Make routes learned over MLAG Peer-link less preferred on spines to ensure optimal routing
   set origin incomplete
!
router bgp 65102
   bgp log-neighbor-changes
   router-id 10.255.0.5
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   !
   neighbor IPv4-UNDERLAY-PEERS peer group
   neighbor IPv4-UNDERLAY-PEERS password 7 7x4B4rnJhZB438m9+BrBfQ==
   neighbor IPv4-UNDERLAY-PEERS send-community
   neighbor IPv4-UNDERLAY-PEERS maximum-routes 12000
   !
   neighbor MLAG-IPv4-UNDERLAY-PEER peer group
   neighbor MLAG-IPv4-UNDERLAY-PEER remote-as 65102
   neighbor MLAG-IPv4-UNDERLAY-PEER next-hop-self
   neighbor MLAG-IPv4-UNDERLAY-PEER description dc1-leaf2b
   neighbor MLAG-IPv4-UNDERLAY-PEER route-map RM-MLAG-PEER-IN in
   neighbor MLAG-IPv4-UNDERLAY-PEER password 7 4b21pAdCvWeAqpcKDFMdWw==
   neighbor MLAG-IPv4-UNDERLAY-PEER send-community
   neighbor MLAG-IPv4-UNDERLAY-PEER maximum-routes 12000
   neighbor 10.255.1.97 peer group MLAG-IPv4-UNDERLAY-PEER
   neighbor 10.255.1.97 description MLAH Peer
   !
   neighbor 10.255.255.8 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.8 remote-as 65100
   neighbor 10.255.255.8 description dc1-spine1
   neighbor 10.255.255.10 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.10 remote-as 65100
   neighbor 10.255.255.10 description dc1-spine2
   redistribute connected route-map RM-CONN-2-BGP
   !
   address-family ipv4
      no neighbor EVPN-OVERLAY-PEERS activate
      neighbor IPv4-UNDERLAY-PEERS activate
      neighbor MLAG-IPv4-UNDERLAY-PEER activate
```

### EVPN

```
router bgp 65102
   neighbor EVPN-OVERLAY-PEERS peer group
   neighbor EVPN-OVERLAY-PEERS update-source Loopback0
   neighbor EVPN-OVERLAY-PEERS bfd
   neighbor EVPN-OVERLAY-PEERS ebgp-multihop 3
   neighbor EVPN-OVERLAY-PEERS password 7 Q4fqtbqcZ7oQuKfuWtNGRQ==
   neighbor EVPN-OVERLAY-PEERS send-community
   neighbor EVPN-OVERLAY-PEERS maximum-routes 0
   neighbor 10.255.0.1 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.1 remote-as 65100
   neighbor 10.255.0.1 description dc1-spine1_Loopback0
   neighbor 10.255.0.2 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.2 remote-as 65100
   neighbor 10.255.0.2 description dc1-spine2_Loopback0
   !
   address-family evpn
      neighbor EVPN-OVERLAY-PEERS activate
   !
   vlan-aware-bundle VRF-10
      rd 10.255.0.5:10
      route-target both 10:10
      redist learned
      vlan 10-49
   !
   vlan-aware-bundle VRF-11
      rd 10.255.0.5:11
      route-target both 11:11
      redist learned
      vlan 50-99
   !
   vrf VRF10
      rd 10.255.0.5:10
      route-target both 10:10
      router-id 10.255.0.5
      no neighbor 10.255.1.96
      no neighbor 10.255.1.96 desc
      neighbor 10.255.1.97 peer group MLAG-IPv4-UNDERLAY-PEER
      redistribute connected route-map RM-CONN-2-BGP-VRFS
```

### VXLAN

```
vrf instance VRF10
vrf instance VRF11
ip routing vrf VRF10
ip routing vrf VRF11
!
vlan 10
   name VRF10_VLAN10
!
interface Vlan11
   description VRF10_VLAN10
   no shutdown
   vrf VRF10
   ip address virtual 10.10.10.1/24
!
vlan 11
   name VRF10_VLAN11
!
interface Vlan11
   description VRF10_VLAN11
   no shutdown
   vrf VRF10
   ip address virtual 10.10.11.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan virtual-router encapsulation mac-address mlag-system-id
   vxlan udp-port 4789
   vxlan vlan 10-1000 vni 10010-11000
   vxlan vrf VRF10 vni 10
   vxlan vrf VRF11 vni 11
```

## SVC Leaf B

### Underlay Interfaces

```
interface Loopback0
   description ROUTER_ID
   no shutdown
   ip address 10.255.0.6/32
!
interface Loopback1
   description VXLAN_TUNNEL_SOURCE
   no shutdown
   ip address 10.255.1.5/32
!
interface Ethernet1
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.13/31
!
interface Ethernet2
   no shutdown
   mtu 9124
   no switchport
   ip address 10.255.255.15/31
```

### MLAG

```
vlan 4093
   name MLAG_L3
   trunk group MLAG
!
vlan 4094
   name MLAG
   trunk group MLAG
!
interface Vlan4093
   description MLAG_L3
   no shutdown
   mtu 9124
   ip address 10.255.1.97/31
!
interface Vlan4094
   description MLAG
   no shutdown
   mtu 9124
   no autostate
   ip address 10.255.1.65/31
!
interface Ethernet3
   no shutdown
   channel-group 3 mode active
!
interface Ethernet4
   no shutdown
   channel-group 3 mode active
!
interface Port-Channel3
   description MLAG Po
   no shutdown
   switchport mode trunk
   switchport trunk group MLAG
   switchport
!
ip virtual-router mac-address 00:1c:73:00:00:99
mlag configuration
   domain-id DC1_L3_LEAF2
   local-interface Vlan4094
   peer-address 10.255.1.64
   peer-link Port-Channel3
   reload-delay mlag 300
   reload-delay non-mlag 330
```

### BGP

```
service routing protocols model multi-agent
ip routing
!
ip prefix-list PL-LOOPBACKS-EVPN-OVERLAY
   seq 10 permit 10.255.0.0/27 eq 32
   seq 20 permit 10.255.1.0/27 eq 32
!
ip prefix-list PL-MLAG-PEER-VRFS
   seq 10 permit 10.255.1.96/31
!
route-map RM-CONN-2-BGP permit 10
   match ip address prefix-list PL-LOOPBACKS-EVPN-OVERLAY
!
route-map RM-CONN-2-BGP-VRFS deny 10
   match ip address prefix-list PL-MLAG-PEER-VRFS
!
route-map RM-CONN-2-BGP-VRFS permit 20
!
route-map RM-MLAG-PEER-IN permit 10
   description Make routes learned over MLAG Peer-link less preferred on spines to ensure optimal routing
   set origin incomplete
!
router bgp 65102
   bgp log-neighbor-changes
   router-id 10.255.0.6
   no bgp default ipv4-unicast
   maximum-paths 4 ecmp 4
   !
   neighbor IPv4-UNDERLAY-PEERS peer group
   neighbor IPv4-UNDERLAY-PEERS password 7 7x4B4rnJhZB438m9+BrBfQ==
   neighbor IPv4-UNDERLAY-PEERS send-community
   neighbor IPv4-UNDERLAY-PEERS maximum-routes 12000
   !
   neighbor MLAG-IPv4-UNDERLAY-PEER peer group
   neighbor MLAG-IPv4-UNDERLAY-PEER remote-as 65102
   neighbor MLAG-IPv4-UNDERLAY-PEER next-hop-self
   neighbor MLAG-IPv4-UNDERLAY-PEER description dc1-leaf2
   neighbor MLAG-IPv4-UNDERLAY-PEER route-map RM-MLAG-PEER-IN in
   neighbor MLAG-IPv4-UNDERLAY-PEER password 7 4b21pAdCvWeAqpcKDFMdWw==
   neighbor MLAG-IPv4-UNDERLAY-PEER send-community
   neighbor MLAG-IPv4-UNDERLAY-PEER maximum-routes 12000
   neighbor 10.255.1.96 peer group MLAG-IPv4-UNDERLAY-PEER
   neighbor 10.255.1.96 description MLAG Peer
   !
   neighbor 10.255.255.12 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.12 remote-as 65100
   neighbor 10.255.255.12 description dc1-spine1
   neighbor 10.255.255.14 peer group IPv4-UNDERLAY-PEERS
   neighbor 10.255.255.14 remote-as 65100
   neighbor 10.255.255.14 description dc1-spine2
   redistribute connected route-map RM-CONN-2-BGP
   !
   address-family ipv4
      no neighbor EVPN-OVERLAY-PEERS activate
      neighbor IPv4-UNDERLAY-PEERS activate
      neighbor MLAG-IPv4-UNDERLAY-PEER activate
```

### EVPN

```
router bgp 65102
   neighbor EVPN-OVERLAY-PEERS peer group
   neighbor EVPN-OVERLAY-PEERS update-source Loopback0
   neighbor EVPN-OVERLAY-PEERS bfd
   neighbor EVPN-OVERLAY-PEERS ebgp-multihop 3
   neighbor EVPN-OVERLAY-PEERS password 7 Q4fqtbqcZ7oQuKfuWtNGRQ==
   neighbor EVPN-OVERLAY-PEERS send-community
   neighbor EVPN-OVERLAY-PEERS maximum-routes 0
   neighbor 10.255.0.1 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.1 remote-as 65100
   neighbor 10.255.0.1 description dc1-spine1_Loopback0
   neighbor 10.255.0.2 peer group EVPN-OVERLAY-PEERS
   neighbor 10.255.0.2 remote-as 65100
   neighbor 10.255.0.2 description dc1-spine2_Loopback0
   !
   address-family evpn
      neighbor EVPN-OVERLAY-PEERS activate
   !
   vlan-aware-bundle VRF-10
      rd 10.255.0.6:10
      route-target both 10:10
      redist learned
      vlan 10-49
   !
   vrf VRF10
      rd 10.255.0.6:10
      route-target both 10:10
      router-id 10.255.0.6
      no neighbor 10.255.1.95
      no neighbor 10.255.1.95 desc
      neighbor 10.255.1.96 peer group MLAG-IPv4-UNDERLAY-PEER
      neighbor 10.255.1.96 description dc1-leaf2a_Vlan3009
      redistribute connected route-map RM-CONN-2-BGP-VRFS
```

### VXLAN

```
vrf instance VRF10
vrf instance VRF11
ip routing vrf VRF10
ip routing vrf VRF11
!
vlan 10
   name VRF10_VLAN10
!
interface Vlan10
   description VRF10_VLAN10
   no shutdown
   vrf VRF10
   ip address virtual 10.10.10.1/24
!
vlan 11
   name VRF10_VLAN11
!
interface Vlan11
   description VRF10_VLAN11
   no shutdown
   vrf VRF10
   ip address virtual 10.10.11.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan virtual-router encapsulation mac-address mlag-system-id
   vxlan udp-port 4789
   vxlan vlan 10-1000 vni 10010-11000
   vxlan vrf VRF10 vni 10
   vxlan vrf VRF11 vni 11
```

## Leaf 2A

### MLAG

```
vlan 10
   name VLAN10
!
vlan 11
   name VLAN 11
!
vlan 4094
   name MLAG
   trunk group MLAG
!
interface Vlan4094
   description MLAG
   no shutdown
   mtu 9124
   no autostate
   ip address 10.255.1.64/31
!
mlag configuration
   domain-id DC1_L3_LEAF2
   local-interface Vlan4094
   peer-address 10.255.1.65
   peer-link Port-Channel3
   reload-delay mlag 300
   reload-delay non-mlag 330
!
int e1
   channel-group 1 mode active
   no shut
int e3-4
   channel-group 1 mode active
!
int po1
   switchport
   switchport mode trunk
   switchport trunk
   mlag 1
   no shut
!
int po3
   switchport
   switchport mode trunk
   no shut
```

## Leaf 2B

### MLAG

```
vlan 10
   name VLAN10
!
vlan 11
   name VLAN 11
!
vlan 4094
   name MLAG
   trunk group MLAG
!
interface Vlan4094
   description MLAG
   no shutdown
   mtu 9124
   no autostate
   ip address 10.255.1.65/31
!
mlag configuration
   domain-id DC1_L3_LEAF2
   local-interface Vlan4094
   peer-address 10.255.1.64
   peer-link Port-Channel3
   reload-delay mlag 300
   reload-delay non-mlag 330
!
int e1
   channel-group 1 mode active
   no shut
int e3-4
   channel-group 3 mode active
!
int po1
   switchport
   switchport mode trunk
   switchport trunk
   mlag 1
   no shut
!
int po3
   switchport
   switchport mode trunk
   no shut
```

## Show Commands

```
show ip virtual-router vrf VRF-C
sh vxlan address-table
sh vxlan flood vtep
sh bgp evpn
sh mlag
sh mlag config-sanity
show bgp evpn instance
show bgp evpn route-type ethernet-segment
```

## VARP

* ip virtual-router address is meant for non-VxLAN implementation of switches in MLAG
* The ‘ip virtual-router address’ command requires an IP address to be configured on the SVI where it is applied.
* the ”ip address virtual” command is specific to a VXLAN routing deployment and is used to conserve IP address space

## Multi Homing

### Host 1

```
vlan 301

interface Port-Channel1
description Ethernet Segment Link PortChannel to Host 3
 switchport
 switchport access vlan 301
 no shut

interface Ethernet1
 channel-group 1 mode active

interface Ethernet2
 channel-group 1 mode active

interface Vlan301
 no autostate
 ip address 172.16.31.10/24
!
```

### Host 2

```
vlan 401

interface Port-Channel1
description Ethernet Segment Link PortChannel to Host 3
 switchport
 switchport access vlan 401
 no shut

interface Ethernet1
 channel-group 1 mode active

interface Ethernet2
 channel-group 1 mode active

interface Vlan401
 no autostate
 ip address 172.16.41.10/24
```

### Leaf 1

```
vlan 301

interface Port-Channel1
 switchport
 switchport mode access
 switchport access vlan 301
 no shut
 evpn ethernet-segment
   identifier 0050:0c00:0700:0700:aaaa
   route-target import 00:1c:73:f5:f7:6c
   lacp system-id 0000.0000.aaaa

interface Ethernet7
 channel-group 1 mode active
```

## MLAG

```
 vlan 4094
  name MLAG
  trunk group MLAGVLAN

 spanning-tree mode mstp
 no spanning-tree vlan-id 4094

 ip virtual-router mac-address 001c.7300.0099

interface Vlan4094
  ip address 172.16.255.1/30
  no autostate

interface Port-Channel100
  switchport
  switchport mode trunk
  switchport trunk group MLAGVLAN

interface Ethernet1
  channel-group 100 mode active

interface Ethernet2
  channel-group 100 mode active

mlag configuration
  domain-id MLAGDomainLeaf1Leaf2
  local-interface Vlan4094
  peer-address 172.16.255.2
  peer-link Port-Channel100
  heartbeat-interval 2500
  ```

## VARP

### Spine 1

```
ip routing

ip virtual-router mac-address 001c.7300.0099

interface vlan 10
  ip address 192.168.10.1/24
  ip virtual-router address 192.168.10.254
  no autostate

interface vlan 20
  ip address 192.168.20.1/24
  ip virtual-router address 192.168.20.254
  no autostate

interface vlan 30
  ip address 192.168.30.1/24
  ip virtual-router address 192.168.30.254
  no autostate

interface vlan 40
  ip address 192.168.40.1/24
  ip virtual-router address 192.168.40.254
  no autostate
```

### Spine 2

```
ip routing

ip virtual-router mac-address 001c.7300.0099

interface vlan 10
  ip address 192.168.10.2/24
  ip virtual-router address 192.168.10.254
  no autostate

interface vlan 20
  ip address 192.168.20.2/24
  ip virtual-router address 192.168.20.254
  no autostate

interface vlan 30
  ip address 192.168.30.2/24
  ip virtual-router address 192.168.30.254
  no autostate

interface vlan 40
  ip address 192.168.40.2/24
  ip virtual-router address 192.168.40.254
  no autostate
```