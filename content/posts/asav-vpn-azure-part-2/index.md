---
title: "Using ASAv as a VPN Concentrator in Azure (Part 2)"
date: 2025-10-07
series: ["ASAv in Azure for VPN"]
series_order: 2
categories: ["Networking", "Cloud"]
tags: ["Cisco", "ASA", "Azure", "IPSEC", "Fortinet"]
---

## Overview

In Part 1, I managed to get the ASAv built and BGP peer it with Azure. In Part 2, I'll be creating route-based and policy-based site-to-site VPN tunnels and validating forwarding.

## Route-Based VPN with Fortigate

### Fortigate Side

Since I've been working with Fortgates some, I'll use that for one side of this site-to-site tunnel. I start by creating a an IPSEC tunnel on the Fortigate. I put in the public address that is assigned to the ASAv's Nic1, which corresponds to outside. I've also enabled NAT-T since both my Fortigate and the ASAv outside interfaces sit behind NAT.

![](/images/fg-ipsec-tun-basic.png)

Next, I set IKE to version 2 and slap in my pre-shared key.

![](/images/fortigate-ikev2.png)

For Phase 1, I went with AES256 encryption and SHA256 authentication with DH 20. Not exactly quantum safe, but what are you gonna do?

![](/images/fortigate-phase1.png)

For Phase 2, I used AES256 and SHA512 witih DH 21.

![](/images/fortigate-phase2.png)

The Local and Remote Addresses are address objects I've defined which refer to the LAN behind the Fortigate and a subnet with my test VM in Azure, respectively.

![](/images/fortigate-address-objs.png)

Upon creating that VPN config, the Fortigate creates a Tunnel Interface underneath the WAN interface that it was tied to. In my case, this is WAN1. It will initially have no addressing assigned, so I gave it an APIPA /30, the other address of which will be used on the ASAv ```tunnel1``` interface.

![](/images/fortigate-tunnel-created.png)

![](/images/fortigate-tunnel-int.png)

Next up I added a static route on the Fortigate towards an address group containing that same 10.100.1.0/24 prefix where my test VM in Azure lives. The interface used is the Tunnel interface I was just working with.

![](/images/fortigate-static-route.png)

And finally, I added firewall policy to permit bi-dreictional traffic between the Fortigate LAN subnet and the Azure subnet.

![](/images/fortigate-fw-policy.png)

### ASAv Side

#### Phase 1 IKEv2 Policy

```
crypto ikev2 policy 1
 encryption aes-256
 integrity sha256
 group 20
 prf sha256
 lifetime seconds 86400
crypto ikev2 enable outside
```

#### Phase 2 IPSEC Policy

```
crypto ipsec ikev2 ipsec-proposal Tun-Prop
 protocol esp encryption aes-256
 protocol esp integrity sha-512

crypto ipsec profile Tun-Prof
 set ikev2 ipsec-proposal Tun-Prop
 set pfs group21
 set security-association lifetime seconds 3600

crypto map outside_map 100 match address acl-vpn-l2l
crypto map outside_map 100 set peer <Fortigate Outside IP>
crypto map outside_map 100 set ikev2 ipsec-proposal AES256
crypto map outside_map 100 set security-association lifetime seconds 28800

tunnel-group <Fortigate Outside IP> type ipsec-l2l
tunnel-group <Fortigate Outside IP> general-attributes
 default-group-policy Tun-Grp-Pol
tunnel-group <Fortigate Outside IP>ipsec-attributes
 ikev2 remote-authentication pre-shared-key *****
 ikev2 local-authentication pre-shared-key *****

```

#### Tunnel Interface

```
interface Tunnel1
 nameif FG-Tunnel
 ip address 169.254.2.1 255.255.255.252
 tunnel source interface outside
 tunnel destination <Fortigate Outside IP>
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile Tun-Prof
```

### Objects & Groups

```
object network obj-local-net1
 subnet 10.100.1.0 255.255.255.0
object network obj-craig-lan
 subnet 172.20.109.0 255.255.255.0
object-group network obj-grp-local-nets
 network-object object obj-local-net1
object-group network obj-grp-craig-nets
 network-object object obj-craig-lan
```

#### Access List

```
access-list acl-vpn-l2l extended permit ip object-group obj-grp-local-nets object-group obj-grp-craig-nets
```

#### Disable NAT for Tunneled Traffic

```
nat (inside,outside) source static obj-grp-local-nets obj-grp-local-nets destination static obj-grp-craig-nets obj-grp-craig-nets no-proxy-arp route-lookup
```

#### Add and Redistribute Static Route

```
route FG-Tunnel 172.20.109.0 255.255.255.0 169.254.2.2 1
router bgp 65501
  address-family ipv4 unicast
  redistribute static
```

## Verification

### From ASAv

![](/images/asav-fortigate-phase1.png)

![](/images/asav-fortigate-phase2.png)

### From Azure

![](/images/azaure-asav-routes.png)

### Client behind Fortigate

![](/images/trace-fg-asav.png)

## Conclusion

In this short series, I deployed an ASAv inside of Azure in order to terminate site-to-site VPN tunnels, rather than using the native Azure VPN gateway option. This lets me monitor and troubleshoot tunnels with more familiarity and visibility than I've been able to get with Azure itself. 

My test tunnel was built as a route-based VPN. On the Azure side, I created a static route to the subnet behind the test branch, and then redistributed that into the BGP process which is peered with an Azure vHub. That allowed the branch LAN prefix to propogate all the way into my Azure routing table, and the traceroute confirms bi-directional traffic between my local test client (172.20.109.100) and my Azure test VM at 10.100.1.4.