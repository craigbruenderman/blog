---
title: "Fortinet Intro - Part 2"
date: "2025-09-02"
draft: true
series: ["A look at Fortinet"]
series_order: 2
categories: ["Networking"]
tags: ["Fortinet", "AWS"]
---

## Overview

Part 2 picks up continuing to the day-0 process of on-boarding Fortigates and FortiSwitches. Here I'll circle back to the FortiManager in AWS deployment I glossed over in Part 1.

### FortiGate Cloud != FortiCloud

Before revisiting FortiManager, FortiGate Cloud is another product I wanted to mention. I'm not using it so I don't have direct experience, but it seems like a similar / competing product with FortiManager + FortiAnalyzer, seemingly for smaller scale and non multi-tenant deployments of Fortigates, FortiAPs, FortiSwitches, and FortiExtenders. I'm looking for a product comparison document between the two approaches if anyone has a link. The naming of this product caused some confusion for me as compared to Forticloud.

[FortiCloud](https://forticloud.com) seems to colloquially mean the whole collection of portal-based account, support, and entitement management for Fortinet products. The [URL](https://forticloud.com) itself takes you to the Asset Management portal, with the Support Portal available in the top menu. I need a Forti-glossary.

## FortiManager in AWS

I'm first testing the self-hosted version of FortiManager, and I'm running it in ~~the best~~ my preferred public cloud, AWS. Not much is needed as far as AWS resources. I reused an existing Lab VPC that already had an external subnet (20) with its default route pointing at an IGW.

![](/images/fortimanager-aws-vpc.png)

Then I followed the [admin guide](https://docs.fortinet.com/document/fortimanager-public-cloud/7.6.0/aws-administration-guide/391685/deploying-fortimanager-on-aws) and spun FortiManager 7.4.7 from ami-0868a4fb8246783c4 in the marketplace, along with with an appropriate Secuirty Group.

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