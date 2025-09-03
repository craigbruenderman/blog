---
title: "Fortinet Intro - Part 1"
date: "2025-08-15"
draft: false
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
| 1 | Cloud Hosted FortiManager | SaaS hosted by Fortinet
| 1 | Self-hosted FortiManager | Running in AWS

### FortiManager

FortiManager is a central management platform for FortiGate firewalls switches, and wireless access points. The console for managing security policies, configurations, and deployment across large or distributed networks. Key features include Zero-Touch Provisioning (ZTP), automated policy provisioning, template-based deployment, advanced automation, and integration with Fortinet's Security Fabric.

#### Features

* Centralized Management: WebUI for managing multiple Fortinet devices, simplifying administration and improving operational efficiency.
* Policy and Configuration Management: Administrators can create, manage, and deploy security policies and configurations across all managed devices from a central location.
* Device Management: Tools for managing device inventory, firmware updates, and individual device configurations.
* Real-time Monitoring: Real-time monitoring of Fortinet devices, including resource usage, network status, and security events.
* Automation and Orchestration: Supports automation and orchestration of security tasks.
* SD-WAN Management: Manages SD-WAN deployments, including configuration, monitoring, and policy enforcement.
* ZTNA and SASE Support: Can be used to manage Zero Trust Network Access (ZTNA) and Secure Access Service Edge (SASE) deployments.
* Role-based Access Control: Supports role-based access control, allowing organizations to assign specific permissions and responsibilities to different users.
* Form Factors: On-prem hardware appliance, On-prem virtual machine, Cloud-based service

### FortiAnalyzer

FortiAnalyzer is a network security logging, analytics, and reporting solution that centralizes security data from FortiGates and third-party devices to provide visibility, threat intelligence, and operational efficiency. It offers real-time and historical analysis through dashboards, enables automated incident response with playbooks, and helps with forensics and compliance.

#### Features

* Centralized Logging & Analytics: Collects security logs from Fortinet and third-party devices to provide a single, consolidated view of the security posture.
* Real-time Visibility: Delivers actionable insights through dashboards and reports, helping security teams identify and react to threats quickly.
* Threat Intelligence & Detection: Uses AI-driven analytics to correlate events, identify known threats, and detect outbreaks, helping to detect and analyze threats across the network.
* Security Automation: Includes built-in playbooks and automation tools to streamline the incident response lifecycle.
* Forensic & Compliance Reporting: Captures detailed data for forensic purposes and provides reports for compliance with privacy policies.
* Asset Identity & Vulnerability Tracking: Displays all devices within the network, including information on known vulnerabilities and associated identities.
* Scalability: Designed to support thousands of devices and can scale storage based on retention and compliance needs.
* Integration: Integrates with the Fortinet Security Fabric and offers connectors for third-party security tools.

### FortiGate & FortiWifi

Fortinet has a huge range of models in its next-gen firewall portfolio, from the dimunitive FG-30G to the 365lb FG-7081 with 16x400Gb interfaces. There are also several [models](https://www.fortinet.com/products/next-generation-firewall#:~:text=Use%20Cases-,Models%20and%20Specs,-Resources) with on-board WiFi for SOHO and retail type applications.

### FortiSwitch

I was not aware how big the [switching portfolio](https://www.fortinet.com/products/ethernet-switches) actually is, and was assuming they would all fall in the access layer and small distribution category and have typical campus networking features. I found there are actuallly 50 available models of FortiSwitches, including a data center line with 32x40/100G interfaces, EVPN VXLAN, 802.1Qbb PFC, PTP, IS-IS etc. My evaluation for now will be for small campus/access applications though, so we'll have to save EVPN for another day.

One of the more relevant properties of access layer switches is their power and PoE capabilities. Here's a table showing those for the 100, 400, and 1000 series switches. For field swappable redundant PSUs, you have to step up to the 1000-series. The [ordering guide](https://www.fortinet.com/content/dam/fortinet/assets/data-sheets/og-fortiswitch.pdf) has more detail on model specifics.

![](/images/fortiswitch-poe-table.png)

FortiSwitches have some interfaces pre-configured as FortiLink interfaces, which I'll dive into more later. For now, let's call it a convenience feature for FortiGates to adopt and manage other Forti-products using a proprietary protocol something like CAPWAP.

## Getting Started

I have to admit I was lost when starting from scratch with this equipment. Much of my confusion stemmed from not having a crash course in Forti-dialect. Since the equipment I'm working with is evaluation gear, some of the processes differ from what a normal commercial customer would experience as well. As with most networking equipment these days, there are administrative provisioning steps involved prior to getting on with the technical configuration, and these vary widely between vendors, and sometimes even within individual portfolios of a single vendor. For Fortinet, and in my specific case of evaluation gear, it looks like this:

0. Account Creation
1. Product Registration
2. Receive equipment and evaluation licensing keys
3. Register devices to your customer portal via method of choice
4. Boot strap devices
5. On-board devices into FortiManager (if applicable)
6. Configure and operate devices

### Account Creation

You may already have this if you've worked with Fortinet in the past. If you're a new Fortinet customer, you'll need to create an account. This is a self-service step [here](https://support.fortinet.com). I would suggest enable 2FA in the Security Credentials section of your profile settings as I've had trouble receiving one-time codes by email before the SAML request times out. I used Google Authenticator.

### Procurement

This is where customers work with the Fortinet and reseller account teams on paperwork to procure hardware and short-term evaluation licensing and support. In my evaluation case, this was just filling out a set of forms with ship-to and partner information. Obviously there would be a purchase process for production scenarios.

### Fulfillment

Product is shipped and Service Entitlement documents are emailed. I received about a dozen documents for my evaluation which was confusing at first. My understanding is that this process can vary, depending on whether additional deployment logistics services have been arranged, such as [FortiDeploy](https://www.fortinet.com/content/dam/fortinet/assets/data-sheets/FortiDeploy.pdf). Here's an example of what these PDFs look like.

![](/images/fortinet-service-entitlement-example.png)

### Product Registration

This might be better named / thought of as "claiming" as opposed to "registering", but essentially this is the step where device serial numbers get associated with Fortinet portal customer accounts. The ad hoc method to register/claim a product is to click Register More, input the serial of the product, and choose whether you're gov or non-gov.

![](/images/fg-reg-serial.png)

Then enter the FORTICLOUD KEY found on a sticker on top of your device, along with the purchase partner. Description and contract are optional here, as iss the asset folder which can be used for logical organization.

![](/images/fg-reg-forticloud-key.png)

### Device Bootstrapping

The next step is to get your Fortigate online so that it can be managed ad-hoc or, in my case, registered with FortiManager. For the FortiGate 50G's I'm testing, this is as simple as feeding a their WAN interface a link that provides DHCP and a path to the Internet. If the available Internet link doesn't provide DHCP, cable into one of the Fortigates LAN ports and pull DHCP. The Fortigate admin interface will be at https://192.168.1.99 with username **admin** and an empty password. On Fortigates with on-board WiFi, they will broadcast the SSID **fortinet**, with PSK **fortinet** which can be used instead of cabling to a LAN interface.

### FortiManager On-boarding

For multi-device management, templating, and other benefits, I'm using FortiManager as an element manager. I'll cheat a bit here and continue on with a FortiManager instance I've already built on AWS, but having a functional FortiManager is an obvious pre-req if you're going to use it. As mentioned, FortiManager can be consumed as a SaaS from Fortinet themselves, but it does have [some limitations](https://docs.fortinet.com/document/fortimanager-cloud/7.6.4/release-notes/865961/limitations-of-fortimanager-cloud). I haven't worked with that offering yet, but I presume it has some commissioning process of its own and then can be used in this step.

To on-board a Fortigate to FortiManager, there is an ad hoc method and a ZTP method. To use the ad hoc method, hit the admin interface mentioned above and select Fabric Connectors from the left nav menu.

![](/images/fg-fortimanager-select.png)

Click into FortiManager and provide the flavor and IP/hostname of your FortiManager instance. This will complete the Fortigate side of on-boarding.

![](/images/fg-fortimanager-config.png)

Finally, login to FortiManager and navigate to Device Manager --> Devices & Groups --> Unauthorized Devices.

![](/images/fg-fortimanager-auth.png)

Choose the correct device and authorize it.

![](/images/fg-fortimanager-auth2.png)

## A word on mounting

The FG-50G's don't include any rack-mounting hardware by default, but they are orderable and I should have my hands on one sometime. This 3rd party [rackmount kit](https://rackmount.it/products/rm-fr-t21?_pos=1&_fid=c8faf3f96&_ss=c) is also available from [Rackmount.IT](https://rackmount.it/). I like that it holds the PSU on models that have use that style of rectifier with a big chuck of electronics in the middle of the span. This kit allows LED side mounting while bringing the interfaces around via included couplers.

## Conclusion

That's a wrap for part 1. In part 2 I'll continue with device and policy configuration and attempt to get ZTP working.