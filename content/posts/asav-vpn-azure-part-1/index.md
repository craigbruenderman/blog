---
draft: false
title: "Using ASAv VPN Concentrator in Azure (Part 1)"
date: 2025-10-02
series: ["ASAv in Azure for VPN"]
series_order: 1
categories: ["Networking", "Cloud", "Automation"]
tags: ["Cisco", "ASA", "Azure", "Terraform"]
---

## Overview

I've seen many struggles with Azure network plumbing, including mysterious instability with their managed VPN gateways. Troubleshooting these issues has been hard, both for the usual reason that IPSEC tunnels are often under two-party adminstration but also because we lose all the usual real-time inspection and troubleshooting tools we're used to when dealing directly with firewall devices. The abstraction that Azure VGW's create becomes a liabilty in this case.

My use case here will be to deploy a single ASAv within Azure to terminate policy-based and route-based LAN to LAN IPSEC VPN tunnels. I'll be BGP peering the ASAv with Azure vWAN and use [Reverse Route Injection](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/sec-vpn/b-security-vpn/m_sec-rev-rte-inject-0.pdf) to advertise the prefixes on the far side of any established tunnels into Azure, and to advertise the prefixes within Azure to the ASA for return routing.

Here's a diagram of the use case I'm building:

![](/images/azure-asav-topo.png)

## ASAv Deployment

Cisco has a ASAv Getting Started Guide doc that I used, but it doesn't cover some critical things that I needed to get my ASAv deployed from the Marketplace. It took me 5 attempts to get a single ASAv successfully deployed. My deploys were failing for a variety of cryptic reasons, and did not clean themselves up, causing subsequent deploys to fail due to existing storage objects.

### Interface Layout

The ASAv in Azure Marketplace wants to use 4 interfaces, mapped like so:

* Mgmt0/0 (Management): Used to connect the ASA virtual via SSH or ASDM
  * Can’t be used for “through” traffic
* Gi0/0 (outside): Used to connect the ASA virtual to the public network
* Gi0/1 (inside): Used for BGP peer with Azure vHub and forwarding into Azure
* Gi0/3 (DMZ): Uunused in my case, but could connect the ASA virtual to the DMZ network when using the c3.xlarge interface

Initially when I deployed, the ASAv installed a default route in the global routing table due to Mgmt0/0 having been set out of the box with ```ip address dhcp setroute``` along with ```no management-only```. I certainly don't want my default route point out the management interface, so that needs fixing. The sequencing on this is important so that I don't knock down my management path to the ASAv. Rather than building a jump box, the easiest appraoch to fixing this is probbaly just using the Azure serial console and applying an initial config.

## Azure Setup

I'm starting by making everything ready in Azure. I stepped through this manually at first, and then translated the process into Terraform once I had my head around it. This builds out the resource dependencies needed in Azure and renders an initial config for the ASAv into a local file called ```asa_config.txt```.

{{< github repo="craigbruenderman/azure-asav" showThumbnail=true >}}

![](/images/asav-tf-apply.png)

### Marketplace Image

I initially wanted to deploy all of this with Terraform, but I couldn't get the module working to deploy an ASAv Marketplace VM, so I ended up doing the ASAv VM deployment manually from the Marketplace. This also meant I couldn't Terraform the Azure BGP peering side of things since the ASAv internal address isn't yet known, but I did go ahead and put some route-map scaffolding in.

With the initial Terraform run, almost everything is in place so the Marketplace deployment is very straightforward.

![](/images/asav-marketplace-deploy.png)

## ASA Config

Once the VM starts, I drop into the Azure Serial Console and apply the initial config so that the ASAv is IP reachable on the public IP for Management0/0.

![](/images/asav-azure-serial-console.png)

Now I can SSH directly to the ASAv to validate routing and perform its VPN config. For reasons I haven't yet figured out, the ASAv installs a default route in the management table just fine, but not in the global table even though I have the ```setroute``` directive on Gi0/0. So I can manage the ASAv ok, but there is no default route pointing out the ```outside``` interface. 

![](/images/asav-routing-tables.png)

Until I understand why, I'll just add a static default towards the gateway of the outside subnet.

```route outside 0.0.0.0 0.0.0.0 10.100.0.33```

![](/images/asav-defaults-fixed.png)

That looks better.

## BGP

The ASAv already has a basic BGP config on it from the initial config I TF generated and pasted. The peer IPs and ASN are corectly populated by the template since they were returned from TF after generating the vWAN and vHUB constructs. In my initial config, I used multihop as required in Azure, and also added static routes to reach the Azure BGP peers via the ```inside``` interface.

![](/images/asav-bgp-config.png)

Now that I know my ASAv inside IP, I build the Azure side of the BGP peering.

![](/images/azure-asav-ebgp-peer.png)

And the waiting begins.

{{< youtubeLite id="uMyCa35_mOg" label="Blowfish-tools demo" >}}

Eventually Azure finishes and the ASAv shows 2 eBGP peers and a handful for NLRI.

![](/images/asav-ebgp-success.png)

## Route Maps

I pre-built route-maps with Terraform in Azure, but I haven't yet applied them. Azure route-maps apply to an entire vNET as it connects to a vHub, rather than an individuial routing peer which is typical. This is why I created a dedicated vNET just for the ASAv. With these empty place-holder route-maps in place, I'll be ready to filter which prefixes go in and out of the ASAv.

![](/images/azure-rmaps-asav.png)

## Conclusion

I'm going to tie off Part 1 here. In Part 2, I'll continue on and get some tunnels up.