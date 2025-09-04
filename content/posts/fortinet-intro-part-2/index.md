---
title: "Fortinet Intro - Part 2"
date: "2025-09-02"
draft: true
series: ["A look at Fortinet"]
series_order: 2
categories: ["Networking"]
tags: ["Fortinet"]
---

## Overview

Part 2 picks up continuing to the day-0 process of on-boarding Fortigates and FortiSwitches. Here I'll circle back to the FortiManager in AWS deployment I glossed over in Part 1.

### FortiGate Cloud

Before revisiting FortiManager, FortiGate Cloud is another product I wanted to mention. I'm not using it so I don't have direct experience, but it seems like a similar / competing product with FortiManager + FortiAnalyzer, seemingly for smaller scale and non multi-tenant deployments of Fortigates, FortiAPs, FortiSwitches, and FortiExtenders. I'm looking for a product comparison document between the two approaches if anyone has a link. The naming of this product caused some confusion for me as compared to Forticloud. [Forticloud](https://forticloud.com) seems to colloquially mean the whole collection of portal-based account and entitement management for Fortinet products. The URL itself takes you to the Asset Management portal, with the Support Portal available in the top menu. I need a Forti-glossary.

### FortiManager in AWS

[Fortimanager Cloud restrictions](https://docs.fortinet.com/document/fortimanager-cloud/7.6.3/release-notes/865961/limitations-of-fortimanager-cloud)

[Activating VM licenses](https://docs.fortinet.com/document/fortimanager/7.6.3/administration-guide/911740/activating-vm-licenses)

https://docs.fortinet.com/document/fortimanager-public-cloud/7.6.0/aws-administration-guide/819045/about-fortimanager-for-aws

## Tasks

* Review and demonstrate ZTP
* Review and configure Provisioning Templates
* Configure initial firmware baseline
* Configure firmware life cycle policies
* Overview of Fortinet SD-WAN mechanics
* Configure SD-WAN spoke/hub
  * Internet transport
  * LTE transport
  * Private transport
* Configure SD-WAN app routing policies
* Configure SD-WAN Internet use, impairment detection, failover
* 