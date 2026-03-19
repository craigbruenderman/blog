---
title: "Fortinet Part 2 - FortiManager Design & Deployment"
date: "2026-03-13"
draft: true
series: ["A look at Fortinet"]
series_order: 2
categories: ["Networking"]
tags: ["Fortinet", "AWS"]
---

## Overview

Part 2 picks up continuing towards the day-0 process of on-boarding Fortigates and FortiSwitches. FortiManager plays a key role in on-boarding and configuration so here I circle back to the FortiManager in AWS deployment glossed over in Part 1.

### FortiGate Cloud != FortiCloud

Before revisiting FortiManager, FortiGate Cloud is another product I wanted to mention. I'm not using it so I don't have direct experience, but it seems like a similar / competing product with FortiManager + FortiAnalyzer, seemingly for smaller scale and non multi-tenant deployments of Fortigates, FortiAPs, FortiSwitches, and FortiExtenders. I'm looking for a product comparison document between the two approaches if anyone has a link. The naming of this product caused some confusion for me as compared to Forticloud.

[FortiCloud](https://forticloud.com) seems to colloquially mean the whole collection of portal-based account, support, and entitement management for Fortinet products. The [URL](https://forticloud.com) itself takes you to the Asset Management portal, with the Support Portal available in the top menu. I need a Forti-glossary.

## FortiManager in AWS

I'm first testing the self-hosted version of FortiManager, and I'm running it in ~~the best~~ my preferred public cloud, AWS.
FortiManager does not sit in the control or data plane of any of the Forti-devices that it will end up managing, it is just the management plane. Still, this is an important function and I want it to be both highly available, and horizontally scalable to a very large number of devices under management. To meet these goals, I combine 2 separate, but sort of related concepts. 

### FortiManager HA (Cluster)

The way to achieve high-availability with self-hosted FortiManager is to deploy 2 or more of them in an [HA cluster](https://docs.fortinet.com/document/fortimanager/latest/administration-guide/568591/high-availability). I'm used to these types of appliances coming in just an active-standby pair, but FortiManager HA clusters can consist of up to 5 units: one primary and up to 4 backups. All units are admin accessible during steady-state, but managed devices only connect with the primary unit. Failure detection and recovery can either be manual (no thanks), or automatic using VRRP. For my purposes, a primary unit with a single backup unit is appropriate and I will split them between AWS AZ's. I suppose some customers had the rquirement for N+4 along the way, but I'm happy enough with AWS EC2's own robustness and will stich with N+1. Ideally the units would be split between AWS Regions, but we'll see later there is a problem with that. So my Terraform will deploy 2 EC2 instances from AMI which I'll configure as a 2-node cluster and that will check the box for high-availability. What about horizontal scale for devices?

### Fabric of FortiManager

A single FortiManager, be it a single unit or HA cluster, has an upper limit of how many Forti-devices it can manage. The absolute number of these seems a bit elusive and I've seen widely differing versions of the number. In the hardware appliance FortiManager world, this is SKU dependent and in the virtual FortiManager world I live in, it is dependent on the underlying VM resources (EC2 in this case). It is also dependent on other variables that I don't have a great understanding of, which is where I think the ambiguity lies. Some of these dependencies are the number of tunnels a FortiManager must maintain to its managed Fortigates, the rate and complexity of of policy pushes, rate of telemetry events coming inbound from Fortigates and their subordinate switches and WAPs, and other items that I'm still researching. But to be sure, there is some upper limit to the number of devices a FortiManager can handle before it becomes problematic. The scale-out technique I'm pursusing is known as [Fabric of FortiManager](https://docs.fortinet.com/document/fortimanager/7.6.5/fabric-of-fortimanager-deployment-guide/821330/introduction), which seems like it's missing an 's' since it supports up to 32 fabric member nodes coordinating to provide horizontal scale. The architecture looks like this:

![](/images/fmg-fabric-arch.png)

There are functionally 2 roles in Fabric of FortiManager, the **Supervisor** and **Members**. The Supervisor role is critical to the functionality of Fabric, with Members being less critical. There is no mechanism in Fabric to detect the failure of a Supervisor and have a Member promoted to take that role, so the strategy instead is to *HA cluster* the Supervisor, and then use stand-alone Members.

Not much is needed as far as AWS resources. I whipped up some Terraform to deploy a topology like this:

![](/images/fortimanager-aws-vpc.png)

I followed the [admin guide](https://docs.fortinet.com/document/fortimanager-public-cloud/7.6.0/aws-administration-guide/391685/deploying-fortimanager-on-aws) and spun FortiManager 7.0.16 from ami-037e81156060a1256 in the marketplace, along with with an appropriate Secuirty Group for SSH and HTTPS management. For some reason I couldn't find a Bring Your Own License version of the AMI with anything newer than 7.0.16, so I just went with it and will upgrade to 7.6.6.

![](/images/fortimanager-aws-instance.png)

After resolving a mixup with my FortiManager registration code, I went back to Forticloud and registered the FortiManager asset which allowed me to download a license key.

![](/images/fortimanager-license-dl.png)

Then I accessed the FortiManager instance via https on it's public EIP, and logged in with **admin** and the instance-id as its default password, which I promptly changed. From this UI, you'll be able to upload the license file and [activate](https://docs.fortinet.com/document/fortimanager/7.6.4/administration-guide/911740) the FortiManager VM.

{{< alert >}}
**Warning!** This is just a lab, but my instance started getting probed quickly via SSH, so mind those Security Groups.
{{< /alert >}}

![](/images/fortimanager-dash.png)

At this point, I have a functional FortiManager, ready to manage Forti-devices.

[Fortimanager Cloud restrictions](https://docs.fortinet.com/document/fortimanager-cloud/7.6.3/release-notes/865961/limitations-of-fortimanager-cloud)

## FortiManaging

## Tasks

* HA setup
```bash
get system ha status
get system status
```
* HA troubleshooting
```bash
execute ha synchronize stop
diagnose debug reset
diagnose debug enable
diagnose debug console timestamp enable
diagnose debug application hasync -1
diagnose debug application hatalk -1
execute ha synchronize start
```

* Review and demonstrate ZTP
* Review and configure Provisioning Templates
  * Problem with config pushes
* Configure initial firmware baseline
* Configure firmware life cycle policies
* Overview of Fortinet SD-WAN mechanics
* Configure SD-WAN spoke/hub
  * Internet transport
  * LTE transport
  * Private transport
* Configure SD-WAN app routing policies
* Configure SD-WAN Internet use, impairment detection, failover
* Problem with DNS lookups when transport is down