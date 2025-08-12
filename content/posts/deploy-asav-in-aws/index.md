---
draft: true
title: "Deploying Cisco ASAv in AWS"
date: 2025-07-06
categories: ["Networking", "Cloud"]
tags: ["Cisco", "ASA", "AWS"]
---

# Overview

I often work with Cisco ASA for on-prem and public cloud situations, so I find myself needing to spin them up and down for lab purposes. This post will show the process I use to deploy ASAv in AWS for testing VPNs or whatever I wish.

## Topology

## AWS Setup

### Bastion Host

I create sections in my ~/.ssh/config file for the bastion and the ASAv so I can reference them conveniently with SSH.

```
#~/.ssh/config

Host aws-bastion
  User ec2-user
  HostName <public_ip_here>

Host aws-workload
        User ec2-user
        HostName <private_ip_here>

Host asav
  User craigb
  HostName <private_ip_here>
```

```
ssh -J ec2-user@public_ip admin@asav_private_ip
```

## ASAv Deployment

Cisco has a decent ASAv Getting Started Guide doc that I used.

### Interface Layout

* DMZ (Optional & unused in my case): Used to connect the ASA virtual to the DMZ network when using the c3.xlarge interface
* Management interface: Used to connect the ASA virtual to the ASDM; can’t be used for “through” traffic
  * Mgmt0/0 in my case
* Inside: Used to connect the ASA virtual to inside hosts
  * Te0/0 in my case
* Outside: Used to connect the ASA virtual to the public network
  * Te0/1 in my case

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