---
draft: true
title: "Using ASAv as a VPN Concentrator in Azure"
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

