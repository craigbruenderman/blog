---
draft: true
title: "Cisco SD-WAN (Part 3)"
date: "2025-07-14"
series: ["Cisco SD-WAN"]
series_order: 3
categories: ["Networking"]
tags: ["Cisco", "SD-WAN"]
---

## Logical Design

Here's the logical design I'll be working with:

![](/images/sdwan-logical.png)

## Onboarding

### Manual

With the manual configuration method, the idea is to configure the minimum network connectivity and the minimum identifying information along with the SD-WAN Validator IP address or hostname. The WAN Edge router attempts to connect to the SD-WAN Validator and discover the other network control components from there. In order for you to bring up the WAN Edge router successfully, there are a few things that need to be configured on the WAN Edge router:

* Configure an IP address and gateway address on an interface connected to the transport network, or alternatively, configure Dynamic Host Configuration Protocol (DHCP) in order to obtain an IP address and gateway address dynamically. The WAN Edge should be able to reach the SD-WAN Validator through the network.
* Configure the SD-WAN Validator IP address or hostname. If you configure a hostname, the WAN Edge router needs to be able to resolve it. You do this by configuring a valid DNS server address or static hostname IP address mapping under VPN 0.
* Configure the organization name, system IP address, and site ID. Optionally, configure the host name.

### ZTP