---
draft: true
title: "Using ASAv as a VPN Concentrator in Azure"
date: 2025-09-18
categories: ["Networking", "Cloud"]
tags: ["Cisco", "ASA", "Azure"]
---

## Overview

I've seen many struggles with Azure network plumbing, including mysterious instability with their managed VPN gateways. Troubleshooting these issues has been hard, both for the usual reason that IPSEC tunnels are typically under two-party control, but also because we lose all the usual real-time inspection and troubleshooting tools we're used to when dealing directly with firewall devices. The abstraction that Azure VGW's create becomes a liabilty in this case.

My use case here will be to deploy a single ASAv within Azure to terminate policy-based and route-based LAN to LAN IPSEC VPN tunnels. I'll be BGP peering the ASAv with Azure vWAN and use [Reverse Route Injection](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/sec-vpn/b-security-vpn/m_sec-rev-rte-inject-0.pdf) to advertise the prefixes on the far side of any established tunnels into Azure, and to advertise the prefixes within Azure to the ASA for return routing.

Here's a diagram of the use case I'm building:

![](/images/azure-asav-topo.png)

## ASAv Deployment

Cisco has a ASAv Getting Started Guide doc that I used, but it doesn't cover some critical things that I needed to get my ASAv deployed from the Marketplace. It took me 5 attempts to get a single ASAv successfully deployed. My deploys were failing for a variety of cryptic reasons, and did not clean themselves up, causing subsequent deploys to fail due to existing storage objects.

### Interface Layout

The ASAv in Azure Marketplace wants to use 4 interfaces, mapped like so:

* Mgmt0/0 (Management): Used to connect the ASA virtual via SSH or ASDM
  * Can’t be used for “through” traffic
* Gi0/0 (outside): Used to connect the ASA virtual to the public network
* Gi0/1 (inside): Used for BGP peer with Azure vHub and forwarding into Azure
* Gi0/3 (DMZ): Uunused in my case, but could connect the ASA virtual to the DMZ network when using the c3.xlarge interface

Initially when I deployed, the ASAv installed a default route in the global routing table due to Mgmt0/0 having been set out of the box with ```ip address dhcp setroute```. I certainly don't want my default route point out the management interface, so that needs fixing. The sequencing on this is important so that I don't knock down my management path to the ASAv.

## Azure Setup

I'm starting by making everything ready in Azure. I stepped through this manually at first, and then translated the process into Terraform once I had my head around it.

### Marketplace Image

az vm image list -o table --publisher cisco --offer cisco-ftdv --all

https://developer.cisco.com/codeexchange/github/repo/gehoumi/terraform-azurerm-marketplace-linux-vm/

### Day-0 Config

```
interface management0/0
management-only
nameif management
security-level 100
ip address dhcp setroute
no shut
!
same-security-traffic permit inter-interface
same-security-traffic permit intra-interface
!
crypto key generate rsa modulus 2048
ssh 0 0 management
ssh timeout 30
! Important to set this
aaa authentication ssh console LOCAL
username admin password <password> privilege 15
username admin attributes
service-type admin
! required config end
! example dns configuration
dns domain-lookup management
DNS server-group DefaultDNS
! where this address is the .2 on your public subnet
name-server 172.19.0.2
ntp server pool.ntp.org
```

### Baseline Config

```
conf t
username craig password [password_here]
username craig attributes
 service-type admin
 ssh authentication publickey [public_key_here]
aaa authentication ssh console LOCAL
debug ssh 10
un all
```

## ASA Config

### Interfaces

### BGP

### VPN

#### Phase 1

crypto map <map-name> set reverse-route

#### Phase 2