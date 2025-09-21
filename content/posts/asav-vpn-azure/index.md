---
draft: true
title: "Using ASAv as a VPN concentrator in Azure"
date: 2025-09-18
categories: ["Networking", "Cloud"]
tags: ["Cisco", "ASA", "Azure"]
---

## Overview

I've seen many struggles with Azure network plumbing, including mysterious instability with their managed VPN gateways. Troubleshooting these issues has been hard, both for the usual reason that IPSEC tunnels are typically under two-party control, but also because we lose all the usual real-time inspection and troubleshooting tools we're used to when dealing directly with firewall devices. The abstraction that Azure VGW's create becomes a liabilty in this case.

## ASAv Deployment

Cisco has a decent ASAv Getting Started Guide doc that I used, but it still took about 5 tries to get a single ASAv successfully deployed. My deploys were failing for a variety of cryptic reasons, and did not clean themselves up, causing subseuqent deploys to fail due to existing storage objects.

### Interface Layout

The ASAv in Azure Marketplace wants to use 4 interfaces, mapped like so:

![](/images/azure-asav-interfaces.png)

* DMZ (Optional & unused in my case): Used to connect the ASA virtual to the DMZ network when using the c3.xlarge interface
* Management interface: Used to connect the ASA virtual to the ASDM; can’t be used for “through” traffic
  * Mgmt0/0 in my case
* Inside: Used to connect the ASA virtual to inside hosts
  * Te0/0 in my case
* Outside: Used to connect the ASA virtual to the public network
  * Te0/1 in my case

## Topology

## Azure Setup

## Day-0 Config

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
! example ntp configuration
name 129.6.15.28 time-a.nist.gov
name 129.6.15.29 time-b.nist.gov
name 129.6.15.30 time-c.nist.gov
ntp server time-c.nist.gov
ntp server time-b.nist.gov
ntp server time-a.nist.gov
Baseline Config
conf t
username craig password [password_here]
username craig attributes
 service-type admin
 ssh authentication publickey [public_key_here]
aaa authentication ssh console LOCAL
debug ssh 10
un all
```

## Baseline Config

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