---
title: Troubleshooting IPSEC VPNs
draft: true
date: 2025-07-04
image: /images/ipsec-logo.png
categories:
  - Networking
  - Troubleshooting
tags:
  - IPSEC
  - VPN
---

## Overview

There is a decent chance that something will go wrong with the initial deployment of a VPN.

<!--more-->

```
debug crypto condition peer <remote_peer-ip>
debug crypto ikev2 platform 255
debug crypto ikev2 protocol 255
debug crypto ipsec 255
```

```
packet-tracer input inside tcp <inside_host_ip> 1025 <remote_host_ip> ssh
```

```
capture ASP_DROPS type asp-drop acl-drop
show capture ASP_DROPS
```