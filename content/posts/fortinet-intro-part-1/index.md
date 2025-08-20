---
title: "Fortinet Into - Part 1"
date: "2025-08-15"
draft: true
series: ["A look at Fortinet"]
series_order: 2
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

FortiManager is a central management platform for FortiGate firewalls switches, and wireless access points. The console for managing security policies, configurations, and deployment across large or distributed networks. Key features include Zero-Touch Provisioning (ZTP), automated policy provisioning, template-based deployment, advanced automation, and integration with Fortinet's Security Fabric.

#### Features

Centralized Management: WebUI for managing multiple Fortinet devices, simplifying administration and improving operational efficiency.

Policy and Configuration Management: Administrators can create, manage, and deploy security policies and configurations across all managed devices from a central location.

Device Management: Tools for managing device inventory, firmware updates, and individual device configurations.

Real-time Monitoring: Real-time monitoring of Fortinet devices, including resource usage, network status, and security events.

Automation and Orchestration: Supports automation and orchestration of security tasks.

SD-WAN Management: Manages SD-WAN deployments, including configuration, monitoring, and policy enforcement.

ZTNA and SASE Support: Can be used to manage Zero Trust Network Access (ZTNA) and Secure Access Service Edge (SASE) deployments.

Role-based Access Control: Supports role-based access control, allowing organizations to assign specific permissions and responsibilities to different users.

#### Form Factors

* On-prem hardware appliance
* On-prem virtual machine
* Cloud-based service

### FortiAnalyzer

FortiAnalyzer is a network security logging, analytics, and reporting solution that centralizes security data from FortiGates and third-party devices to provide visibility, threat intelligence, and operational efficiency. It offers real-time and historical analysis through dashboards, enables automated incident response with playbooks, and helps with forensics and compliance.

#### Features

Centralized Logging & Analytics: Collects security logs from Fortinet and third-party devices to provide a single, consolidated view of the security posture.

Real-time Visibility: Delivers actionable insights through dashboards and reports, helping security teams identify and react to threats quickly.

Threat Intelligence & Detection: Uses AI-driven analytics to correlate events, identify known threats, and detect outbreaks, helping to detect and analyze threats across the network.

Security Automation: Includes built-in playbooks and automation tools to streamline the incident response lifecycle.

Forensic & Compliance Reporting: Captures detailed data for forensic purposes and provides reports for compliance with privacy policies.

Asset Identity & Vulnerability Tracking: Displays all devices within the network, including information on known vulnerabilities and associated identities.

Scalability: Designed to support thousands of devices and can scale storage based on retention and compliance needs.

Integration: Integrates with the Fortinet Security Fabric and offers connectors for third-party security tools.

### FortiGate & FortiWifi

Fortinet has a huge range of models in its next-gen firewall portfolio, from the dimunitive FG-30G to the 365lb FG-7081 with 16x400Gb interfaces. There are also several [models](https://www.fortinet.com/products/next-generation-firewall#:~:text=Use%20Cases-,Models%20and%20Specs,-Resources) with on-board WiFi for SOHO and retail type applications.

### FortiSwitch

## Getting Started

### FortiManager in AWS

[Fortimanager Cloud restrictions](https://docs.fortinet.com/document/fortimanager-cloud/7.6.3/release-notes/865961/limitations-of-fortimanager-cloud)

[Activating VM licenses](https://docs.fortinet.com/document/fortimanager/7.6.3/administration-guide/911740/activating-vm-licenses)

https://docs.fortinet.com/document/fortimanager-public-cloud/7.6.0/aws-administration-guide/819045/about-fortimanager-for-aws

### FortiGate Activation

The FG_50G's don't come with any rackmounting hardware, but this 3rd party [rackmount kit](https://rackmount.it/products/rm-fr-t21?_pos=1&_fid=c8faf3f96&_ss=c) is available from Rackmount.IT. I like that it holds the PSU and allows LED side mounting while bringing the interfaces around via included couplers.