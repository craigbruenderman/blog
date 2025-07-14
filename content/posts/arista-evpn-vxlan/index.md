---
title: "Arista EVPN VXLAN"
draft: true
date: 2025-07-04
categories: ["Networking"]
tags: ["EVPN", "VXLAN", "Arista"]
---

## Verification Commands

### Compare MAC and VXLAN Address Tables

```
leaf1#sh mac address-table dynamic vlan 101
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
 101    001c.73f1.c601    DYNAMIC     Et7        1       0:01:02 ago
 101    001c.73f2.c601    DYNAMIC     Vx1        1       0:00:49 ago
 101    001c.73f3.c601    DYNAMIC     Vx1        1       0:01:02 ago
 101    001c.73f4.c601    DYNAMIC     Vx1        1       0:00:47 ago
Total Mac Addresses for this criterion: 4
```

```
leaf1#sh vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
 101  001c.73f2.c601  EVPN      Vx1  10.12.12.12      1       0:01:09 ago
 101  001c.73f3.c601  EVPN      Vx1  10.13.13.13      1       0:01:22 ago
 101  001c.73f4.c601  EVPN      Vx1  10.14.14.14      1       0:01:07 ago
Total Remote Mac Addresses for this criterion: 3
```

### Display exported Host MACs learned into EVPN

```
leaf1#show l2Rib output vlan 101
001c.73f4.c601, VLAN 101, seq 1, pref 16, evpnDynamicRemoteMac, source: BGP
   VTEP 10.14.14.14
001c.73f1.c601, VLAN 101, seq 1, pref 16, learnedDynamicMac, source: Local Dynamic
   Ethernet7
001c.73f3.c601, VLAN 101, seq 1, pref 16, evpnDynamicRemoteMac, source: BGP
   VTEP 10.13.13.13
001c.73f2.c601, VLAN 101, seq 1, pref 16, evpnDynamicRemoteMac, source: BGP
   VTEP 10.12.12.12
```

```
show bgp evpn route-type mac-ip vni 10101
show bgp evpn route-type mac-ip vni 10101 detail
show bgp evpn route-type imet vni 10101
show bgp evpn route-type imet detail
```