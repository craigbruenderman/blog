---
title: "Meraki vMX in Azure with vWAN and BGP"
date: 2025-07-20
draft: true
categories: ["Cloud", "Networking"]
tags: ["Meraki", "Azure", "BGP", "vWAN"]
---

## Overview

This post explores using the Meraki vMX virtual device to build AutoVPN from branches into Azure. The vMX will only be used for WAN connectivity between branches, and not to provide any filtering services for traffic in/out of Azure on the public side. I'll be demonstrating a single vMX device in a single Azure region. This vMX will BGP peer with Azure vWAN so that it can inject branch prefixes it learns via AutoVPN, and advertise prefixes for resources within Azure out to the branch WAN.

## Meraki vMX Deployment Modes

Meraki MX's have two different deployment modes that can be used depending on your situation.

[Meraki Deployment Modes](https://documentation.meraki.com/MX/Networks_and_Routing/MX_Addressing_and_VLANs)

### Concentrator (a.k.a. Passthrough) Mode

Concentrator mode is what I want here, as I'm using the vMX purely to terminate branch VPN connectivity, and not to filter any traffic in/out of Azure. My VPN topology will be hub and spoke, where branches are explicitly configured to form tunnels to the vMX in Azure, but not between each other.

> As a layer 2 passthrough device
> Choose this option if you simply want to deploy the WAN appliance:
> * In bridge mode for traffic shaping and additional network visibility.
> * As a one-armed VPN concentrator.

### Routed Mode

Routed mode doesn't match my scenario. It's more typically used on the branch side, where you'd want multiple SVIs and Internet perimeter filtering.

> This is the default selection. Choose this option if you want to use the WAN appliance as a layer 7 firewall to isolate and protect LAN traffic from the Internet (WAN). Client traffic to the Internet will have its source IP rewritten to match the WAN IP of the appliance. In this mode, the WAN appliance is generally also the default gateway for devices on the LAN. This section also provides a link to the DHCP settings page.

[Meraki BGP page](https://documentation.meraki.com/MX/Networks_and_Routing/Border_Gateway_Protocol_(BGP))

## Topology

Here is the topology of what I'll be building.

![](/images/vmx-vwan-topology.png)

## Azure Make Ready

In order to deploy a vMX in Azure, we're going to need some Azure pre-requisite resources. I had some of these items (like the vWAN and vHub) in place from previous lab work and created others as needed.

* A Virtual WAN (vWAN) with appropriate, non-overlapping prefix
* A Virtual Hub (vHub) attached to the vWAN above
* A vNet for the vMX to reside in

## Deploy the vMX in Azure

One of the inputs needed when deploying a vMX from the Azure Marketplace is a token. This token is generated via a button within the Meraki Dashboard from the Network in which the vMX will reside.

![](/images/vmx-gen-token.png)

### Deploy vMX from Marketplace

With the token in hand, search the Martplace for Meraki and use the deployment wizard.

![](/images/azure-vmx-create.png)

**Note** the discrepancy in the screenshot. Prefix shows 172.17, but 172.16 was actually used.

The vMX should come online in about 15 minutes. Recall that this vMX is deployed in one-armed concentrator mode, so it has a single interface which both terminates AutoVPN, and faces inward for BGP peering toward vWAN. This interface has an assigned RFC1918 private IP of 172.16.0.4, and is also Source NATted to a global IP of 4.255.17.115. The private IP will be used for BGP peering, and the public IP is what AutoVPN tunnels will be built to.

![](/images/vmx-online.png)

### Connect vMX vNet to vWAN vHub

Now add a connection between the vNet that the vMX lives in, and the vHub. Notice we must select the Meraki Managed Resource Group in order to get access to the corect vNet in the drop-down.

![](/images/vmx-connect-vnet-vhub.png)

## Azure Side Configuration

Azure has set aside 2 address from the vWAN prefix I picked for BGP peering. It will also select AS 65515 for itself by default. This will be used later in the Meraki Dashboard config.

![](/images/vhub-peering-properties.png)

### Add vMX eBGP Peer

Add a single peer for the single vMX using its private IP that we took note of before, and the ASN of choice (65501).

![](/images/add-vmx-bgp-peer.png)

## Meraki Dashboard Configuration

First set this vMX device to be a VPN hub.
.
![](/images/vmx-hub-setting.png)

Next, head over to the Routing config section.

![](/images/vmx-bgp-spot.png)

Enable BGP and add 2 neighbors, one for each of the vWAN peering addresses found before. It is necessary to enable BGP multi-hop since the vMX interface is not directly connected to the vWAN peering entity. If this isn't set, the peers will never come up. I used the multi-hop value of 3, but 2 might be sufficient if you care to test it.

![](/images/vmx-bgp-neighbor-config.png)

## Deploy Azure Test Workload

In order to have an endpoint to test with, I'll create a Ubuntu VM in Azure. I first create and attach a vNET for test VM. I gave this vNet the prefix 10.0.75.0/24.

![](/images/vnet-test-vms.png)

Next the new virtual machine vNet needs to be attached to vWANs vHub. This will trigger propogation of this vNets prefix into vWAN so that it can be advertised to the vMX via BGP.

![](/images/conn-vm-vnet-vhub.png)

Now create the test VM itself, attaching to the correct vNET.

![](/images/create-test-vm.png)

## Meraki Branch Setup

Now I'll configure a test Branch to use the new vMX as an AutoVPN hub and advertise one of its local VLAN prefixes via AutoVPN. This prefix should make its way into the routing table in Azure if things go well.

![](/images/autovpn-branch-setup.png)

![](/images/)
![](/images/)

## Validation

### BGP Peering Confirmation

I did not time this, but the peering was quite slow to establish. I'd give it at least 20 minutes before taking any troubleshooting steps. Eventually both peers came up as seen below.

#### Peer Status from Azure

![](/images/vmx-peer-up-azure.png)

#### Azure Route Tables

Here's a look at the vHub effective routes once AutoVPN is up. Notice that the 10.119.1.0/24 prefix is being learned. This is an SVI from the test branch. Azure identifies that it learned this via BGP AS 65501 and the next hop is the vMX. Looks good.

![](/images/vhub-effective-routes.png)

Looking further, I check the effective routes from the perspective of my Azure test VM. Again, the test branch prefix is there, but the next hop is a bit curious. 

![](/images/test-vm-effective-routes.png)

#### Peer Status from Meraki Dashboard

![](/images/vmx-peer-up-dashboard.png)

Notice the **# Routes** column. This gives a good indication that some NLRI have actually been exchanged between vWAN and the vMX.

### Test ICMP from Meraki Branch