---
title: 'Azure Route-Based VPN to Cisco ASA with NAT (Part 2)'
date: '2024-11-12'
image: /images/ipsec-logo.png
categories:
    - Azure
    - Networking
tags:
    - ASA
    - Azure
    - IPSEC
    - VPN
---

# Part 2: Configuring Route-based VPN with ASA & Azure

## Overview

In Part 1, I reviewed some foundational knowledge for working with IPSEC VPNs. In this part, I’ll actually configure one.

## ASAv Side

In my lab scenario, the ASA I used for one side of the tunnel happens to be an ASAv, and happens to reside in AWS. Read how I deploy these in another post. This has little bearing on the design, and a physical ASA or ASAv running on any hypervisor will work similarly. This is just what I had handy, and I wanted to point it out to avoid any confusion. If this is confusing, forget that any AWS network constructs are involved here, and the big picture becomes an ASA with a workload behind it, tunneling to Azure which has another workload.

![](/images/azure-asav-detail.png)

More Detailed True Picture

If you are using ASAv on AWS, make sure to understand the things ASAv on AWS does not support:

### VPN Configuration

VPN configuration is a 6 step process, which can be thought of as two parts. Steps 1 & 2 are referred to as the “Phase 1” part. Here is where we define how the tunnel will be protected with IKE. Steps 3 – 6 are referred to as the “Phase 2” part, where we define how data in the tunnel is protected with IPSEC.

Looking at examples can be confusing because there are several permutations that one might find. IOS routers and ASAs will always have slightly different syntax. I’ll try to point out where these differences might pop up.

### Phase 1 Part

#### Step 1 – Define Phase 1 IKE policy

IKEv1 and IKEv2 have different syntax. IKEv1 should be considered deprecated at this point and not used, but it still comes up regularly because… inertia. We all should be using IKEv2.

``` 
! IKEv2 crypto policy example
crypto ikev2 policy 1
! encryption
 encryption aes-256
 integrity sha256
 lifetime seconds 86400
! Diffie-Hellman group
 group 20

crypto ikev2 policy 2
 encryption aes-256
 integrity sha256
 lifetime seconds 86400
 group 20

crypto ikev2 enable outside
```

Note about the integrity command: when using AES-GCM: “An authenticated encryption algorithm provides a combined functionality of encryption and integrity. Such algorithms are called combined mode algorithms. The Support of AES-GCM as an IKEv2 Cipher on IOS feature provides the use of authenticated encryption algorithms for encrypted messages in IKEv2 protocol by adding the Advanced Encryption Standard in Galois/Counter Mode (AES-GCM). AES-GCM supports the key size of 128- and 256-bits—AES-GCM-128 and AES-GCM-256. If AES-GCM is the only encryption algorithm, integrity algorithms cannot be added to the proposal.”

#### Step 2 – Define pre-shared key password (or certificate)

Certificate authentication is possible, but in this case I’m just using a pre-shared key. IKEv1 supports only symmetric passwords, while IKEv2 supports asymmetric passwords.

```
tunnel-group <remote-peer-ip> type ipsec-l2l
tunnel-group <remote-peer-ip> ipsec-attributes
  ikev2 remote-authentication pre-shared-key <PSK here>
  ikev2 local-authentication pre-shared-key <PSK here>
```

### Phase 2 Part

#### Step 3 – Define Phase 2 IPSEC policy

```
 crypto ipsec ikev2 ipsec-proposal AES256
  protocol esp encryption aes-256	
  protocol esp integrity sha-256
```

#### Step 4 – Define triggering traffic

Often referred to as “interesting traffic”. There are multiple methods to define this. If you only have a single subnet on the local and remote side, you can use something simple.

```
! Simple option
access-list acl-VPN-Azure extended permit ip <local subnet> <local mask> <remote subnet> <remote mask>
```

If you have many subnets on either side, or just want to plan for such an occurrence, go with object-groups to define things before referring to them in an ACL.

```
! Option using object-groups
object network obj-local-net1
  description A local network behind ASA
  subnet 192.168.1.0 255.255.255.0
object network obj-local-net2
  description Another local network behind ASA
  subnet 192.168.2.0 255.255.255.0

object-group network obj-Local-Nets
  description Group of all local networks behind ASA
  network-object object obj-local-net1
  network-object object obj-local-net2

object network obj-vnet1
  description A network behind remote side of VPN
  subnet 10.0.1.0 255.255.255.0
object network obj-vnet2
  description Another network behind remote side of VPN
  subnet 10.0.2.0 255.255.255.0

object-group network obj-grp-VPN-Azure-Remote-Nets
  description Group of all local networks on remote side
  network-object object obj-vnet1
  network-object object obj-vnet2

! Refer to those object groups in the ACL
access-list acl-azure-vpn extended permit ip object-group obj-Local-Nets object-group obj-grp-VPN-Azure-Remote-Nets
```

#### Step 5 – Wrapping up Step 1 – 4 config in a container

The container used for this purpose is known as a crypto map.

```
! Refer to previously create ACL which defines interesting traffic
crypto map outside_map 101 match address <ACL name>
crypto map outside_map 101 set peer <remote-peer-ip>
crypto map outside_map 101 set ikev2 ipsec-proposal AES256
crypto map outside_map 101 set security-association lifetime seconds 28800
```

#### Step 6 – Apply crypto map to outgoing interface

```
! Attaching to the interface nameif'd outside
crypto map outside_map interface outside
! Assuming this ASA is already performing NAT/PAT
nat (inside,outside) source static obj-Local-Nets obj-Local-Nets destination static obj-grp-VPN-Azure-Remote-Nets obj-grp-VPN-Azure-Remote-Nets
```

## Azure Side

Azure configures differently than the ASA. There are still two basic parts, which are

* VPN Site Creation
* VPN Site Connection

### Azure Side NAT with VWAN

Now that this feature exists, of course there are plenty of caveats. The one that is a real bummer is the requirement for route-based VPNS. I really try to stay away from policy-based VPNs in late 2024, but I still encounter organizations that will not do route-based.

Here are some explanations of Azure-speak NAT terminology.

* **Ingress:** An Ingress SNAT rule maps an On-premises network address space to a translated address space to avoid address Overlap.
* **Egress:** An Egress SNAT rule maps the Azure address space to another translated address space.

For Each NAT rule, following two fields specify the address spaces before and after the translation.

* **Internal Mapping:**  This is the address space before the translation. For an Ingress rule, this field corresponds to the original address space of the On-premises network. For an egress Rule, this is the original Vnet address Space.
* **External Mapping:** This is the address space after the translation for On-premises network (ingress) or Vnet (egress). For Different networks connected to an Azure VPN Gateway, the address spaces for all External Mapping must not overlap with each other and with the network connected without NAT.

A single SNAT rule defines the translation for both directions of a particular network:

* An **IngressSNAT** rule defines the translation of the source IP addresses coming into the Azure VPN gateway from the on-premises network. It also handles the translation of the destination IP addresses leaving from the VNet to the same on-premises network.
* An **EgressSNAT** rule defines the translation of the source IP addresses leaving the Azure VPN gateway to on-premises networks. It also handles the translation of the destination IP addresses for packets coming into the VNet via those connections with the EgressSNAT rule.
* In either case, no DNAT rules are needed.

### If using BGP

Once a NAT rule is defined for a connection, the effective address space for the connection will change with the rule. If BGP is enabled on the Azure VPN Gateway, Select the “Enable BGP Route Translation” to automatically convert the routes learned and advertised on connections with NAT rules:

Learned routes: The destination prefixes of the routes learned over a connection with the Ingress SNAT rules will be translated from the Internal Mapping prefixes (pre-NAT) to the External Mapping prefixes (post-NAT) of those rules.

Advertised routes: Azure VPN gateway will advertise the External Mapping (post-NAT) prefixes of the Egress SNAT rules for the VNet address space, and the learned routes with post-NAT address prefixes from other connections.

BGP peer IP address consideration for a NAT ‘ed on-premises network:

APIPA (169.254.0.1 to 169.254.255.254) address: NAT isn’t supported with BGP APIPA addresses.

Non-APIPA address: Exclude the BGP Peer IP addresses from the NAT range.

The Learned routes on connection without Ingress SNAT rule will not ne converted. The VNet route advertised to connection without Egress SNAT rule will also not be converted.