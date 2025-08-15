---
title: "A Look at Fortinet"
date: "2025-08-07"
draft: true
series: ["Fortinet Intro"]
series_order: 1
categories: ["Networking"]
tags: ["Fortinet"]
---

## Overview

It's been years since I've touched Fortinet in any meaningful way, so I'm taking a fresh look at their portfolio. For some customers, OEMs with strong integrated portfolios are a better fit that picking best of breed products a la carte. I can steel-man both of these arguments, but I'll leave that to the analysts. Suffice to say, Fortinet is the only OEM in the upper-right for wired and wireless LANs, SD-WAN, and SASE, so it's worth a look.

## Components

Here's a list of the kit I've got to evaluate, along with a rundown of each.

| Qty | Device | Notes
| :------- | :------: | :------:
| 2 | FortiWiFi-50G | 
| 2 | FortiGate-50G | 
| 2 | FortiSwitch-124G-FPOE | 
| 1 | Cloud Hosted FortiManager | 
| 1 | Self-hosted FortiManager | Running in AWS


### FortiManager

Centralized Management: WebUI for managing multiple Fortinet devices, simplifying administration and improving operational efficiency.

Policy and Configuration Management: Administrators can create, manage, and deploy security policies and configurations across all managed devices from a central location.

Device Management: Tools for managing device inventory, firmware updates, and individual device configurations.

Real-time Monitoring: Real-time monitoring of Fortinet devices, including resource usage, network status, and security events.

Automation and Orchestration: Supports automation and orchestration of security tasks.

SD-WAN Management: Manages SD-WAN deployments, including configuration, monitoring, and policy enforcement.

ZTNA and SASE Support: Can be used to manage Zero Trust Network Access (ZTNA) and Secure Access Service Edge (SASE) deployments.

Role-based Access Control: Supports role-based access control, allowing organizations to assign specific permissions and responsibilities to different users.

#### Form Factors

* On-prem virtual machine

### FortiAnalyzer

### FortiWifi

### FortiSwitch

## Getting Started

### FortiManager in AWS

https://docs.fortinet.com/document/fortimanager-public-cloud/7.6.0/aws-administration-guide/819045/about-fortimanager-for-aws