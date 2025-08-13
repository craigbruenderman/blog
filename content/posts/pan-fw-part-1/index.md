---
title: "Palo Alto Firewalls in AWS (Part 1)"
draft: false
date: "2025-08-12"
series: ["Palo Alto Firewalls"]
series_order: 1
categories: ["Networking", "Cloud", "Security"]
tags: ["Palo Alto", "AWS"]
---

## Overview

This post will cover basic Palo Alto firewall deployment in AWS for a variety of use cases, be that protecting Internet-facning workloads, terminating IPSEC, or generally labbing scenarios. Once source NAT and filtering is setup, I'll continue on to configure IPSEC along with some source NAT and destination NAT scenarios.

## Topology

Below is the topology I'll be working with and it will evolve as needed if I manage to continue the series to more interesting use cases.

![](/images/aws-pan-base-topology.png)

## AWS Make Ready

Typical AWS networking prerequisites apply here and I'll skip the detailed creation steps.

* VPC in region of choice to house subnets
* Internal subnet to face protected workloads
  * Associated route table
* External subnet to face Internet via IGW
  * Associated route table
* EIP for outside/untrusted interface
* ENI for internal/trusted, external/untrusted, and management interfaces
  * Appropriate ecurity groups for each

### AMI Deployment

![](/images/aws-pan-ec2-1.png)

Start by picking the correct AMI

![](/images/aws-pan-ec2-network.png)

Choose the correct VPC and the right combination of external subnet, public IP assignment and security group with SSH and HTTPS. This is for the management interface, which you may or may not use post-deployment.

These firewall VMs take their sweet time to boot, so give them a good 20 minutes before panic troubleshooting. In particular, there is a period of time where the device will answer SSH connection attempts, but refuse to authenticate with your correct key (which I just went through). Initially access will be via SSH to the assigned public IP, which you'll get form inspecting the ENI attached to the VM, using the keypair you chose.

![](/images/aws-pan-change-admin.png)

Use `set mgt-config users admin password` to change the admin password to something known, and then you'll be able to use the web UI.

![](/images/aws-pan-webui.png)

### Network Interfaces

{{< alert >}}
**Alert** If you intend to utilize VM-Series firewalls behind Amazon ELB Service, you'll want to perform an interface swap before proceeding. Docs [here](https://docs.paloaltonetworks.com/vm-series/11-0/vm-series-deployment/set-up-the-vm-series-firewall-on-aws/about-the-vm-series-firewall-on-aws/management-interface-mapping-for-use-with-amazon-elb)
{{< /alert >}}

Initially, the VM boots with a single elastic network interface, which will get a temporary public IP as it was told. However, a firewall with only a single management interface is not useful. Next we shut down the instance in order to add 2 additional interfaces to it for inside & outside in their respective subnets, and with (likely) wide-open security groups so that the Palo Alto ruleset can see and filter all traffic.

![](/images/aws-pan-eni-attach.png)

I have pre-created the 2 ENI's for inside and outside and named them conspicuously, and here I attach them to the stopped EC2 instance. You'll also notice that I silently created and associated an EIP with the ENI at device index 2, which will map to Palo Alto eth1/2 that I'll use for outside.

![](/images/pan-vm-3-eni.png)

Now I have a problem that I intentionally inflicted because it seems common. There was a warning I ignored when attaching the additional ENIs that looked something like

{{< alert cardColor="#d4d436ff" >}}
**If you attach another network interface to your instance, your current public IP address is released when you restart your instance. Learn more about public IP addresses.**
{{< /alert >}}

So essentially, I've inadvertently removed the public IP from the ENI mapping to the initial Palo Alto management interface. The public IP it had was dynamic and temporary and cannot be reassigned directly. I can't yet manage the PAN via the outside interface because I haven't assigned a management profile to do so. The fix is simple enough - just create and associate to ENI index 0 an additional EIP which can be discarded later if your desire is to manage via the outside interface.

![](/images/aws-pan-rescue-eip.png)

And now we're back in the web UI, albeit on a different public management IP.

## Wrap Up

I'm going to tie this short one off here. In the next part, I'll perform some configuration of the firewall and get an AWS host using it for Internet access. Stay tuned.