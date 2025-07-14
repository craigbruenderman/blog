---
title: "Azure Route-Based VPN to Cisco ASA with NAT (Part 1)"
date: 2024-11-11
categories: ["Networking"]
tags: ["ASA", "Azure", "IPSEC", "VPN"]
---

# Part 1: Concepts for Route-based VPN with ASA &amp; Azure VWAN

## Overview

There are plenty of posts and docs on the topic of building site to site VPNs from Azure to other premise-based firewalls. I’ve done this with a couple different flavors of config on the Azure side, and a few different VPN endpoints on the far side. I recently had my first use-case where I encountered the need to tunnel from Azure to a vendor using ASA policy-based VPN along with requiring me to destination NAT an RFC1918 address of their choosing to the true internal RFC1918 address of the customer. I’ve included concepts and theory up front as a refresher for myself. Skip down to the config section if you’re ready to get on with the build.

This vendor understandably wants to limit how much RFC1918 of its customers it has to keep track of, so they assign small blocks which they don’t use internally for this purpose. This DNAT allows the vendor to source from their global IP addressed systems and cross a VPN towards RFC1918 addresses that they assign for individual customer systems, leaving the customers to address the real systems however they wish. This comes up a frequently, but it’s the first time I had to solve it with the customer systems living in Azure.

I had to dust off a few brain cells on all the topics involved for this multi-part post. For a guy coming from a network background, Azure network plumbing and constructs are bizarre and frustrating. AWS networking also has a learning curve, but is more inline with my way of thinking.

### Simplified Big Picture

![](/images/vpn-nat-simplified.png)

## VPN Concepts

Here is a good overview of IPSEC if you need a primer. I’ve extracted a few slides here for reference. The key to tunnel formation is making sure the proposals match on both ends.

### EIISA

* E – ESP (Encapsulating Security Payload) – (Confidentiality & Authentication)
* I – IKE (Internet Key Exchange)
* I – ISAKMP (Internet SA Key Mgmt Protocol)
* S – SA (Security Association)
* A – AH (Authentication Header) (Integrity & Authentication)

### HADLE

* H – Hash (SHA-256)
* A – Authentication (Preshared Key/Certificate)
* D – Diffie-Hellman (Key Exchange)
* L – Life Time (8hrs)
* E – Encryption (AES-128, 256)

### Encryption Domain & Traffic Selector

These two terms mean roughly the same thing, which is a defined set of network prefixes which you want to traverse the VPN tunnel. I’ll use both terms interchangeably to confuse you reinforce this. These prefixes are used in IKE negotiation for the tunnel. There are two types of traffic selectors. The local traffic selector defines the set of local prefixes from the perspective of the VPN gateway that emits the VPN tunnel. The remote traffic selector defines the set of remote prefixes from the perspective of the VPN gateway that emits the VPN tunnel.

The network prefixes in a traffic selector are typically internal addresses (like RFC1918) which are not reachable on the Internet, but not always. The encryption domain is configured on the devices on each side of a VPN tunnel to direct “interesting” traffic (that which is part of the encryption domain definition) into the VPN tunnel, and uninteresting traffic (that which is not part of the encryption domain definition) elsewhere like directly to the Internet.

Here’s one more description I found. “A traffic selector is an agreement between IKE peers to permit traffic through a VPN tunnel if the traffic matches a specified pair of local and remote addresses. Only the traffic that conforms to a traffic selector is permitted through the associated security association.”

### Route-Based vs. Policy Based VPNs

There are two methods to define the VPN encryption domains, route-based or policy-based. Policy-based vs. route-based VPN configurations differ in how the IPsec traffic selectors are set for a tunnel.

### Policy-Based VPNs

Policy-based VPN devices use the combinations of prefixes from both networks to define how traffic is encrypted/decrypted through IPsec tunnels. This approach is common on older firewalls. IPsec tunnel encryption and decryption are added to the packet filtering and processing engine.

The encryption domain is set to encrypt only specific IP ranges for both source and destination. Policy-based local traffic selectors and remote traffic selectors identify what traffic to encrypt over IPSec. Cisco ASA supports policy-based VPN with crypto maps in version 8.2 and later. If the ASA only supports crypto maps due to code version, then Azure must be configured for route-based with policy-based traffic selectors.

### Route-Based VPNs

In route-based VPNs, the encryption domain / traffic selector is set to allow any traffic which enters the IPSec tunnel. IPSec Local and remote traffic selectors are set to the wildcard 0.0.0.0/0. This means that any traffic routed into the IPSec tunnel is encrypted regardless of the source/destination subnet and routing tables direct traffic to different IPsec tunnels. It’s common on routers platforms where each IPsec tunnel is modeled as a network interface or VTI (virtual tunnel interface). Cisco ASA supports route-based VPNs with the use of Virtual Tunnel Interfaces (VTIs) in versions 9.8+. Cisco Firepower supports route-based VPN with VTIs in versions 6.7+.

This approach is typically used with routers, because routers use Virtual Tunnel Interfaces to terminate VPN tunnels, that way traffic can be routed down various different tunnels based on a destination, (which can be looked up in a routing table). No crypto-maps are used or apply here, and remember that routing takes place prior to NAT.

## ASA Tunnels to Azure

For ASA/FTD configured with a crypto map, Azure must be configured for policy-based VPN or route-based with UsePolicyBasedTrafficSelectors.

The IP addresses range IPSec allows it to participate in the VPN tunnel. The encryption domain is defined with the use of a local traffic selector and remote traffic selector to specify what local and remote subnet ranges are captured and encrypted by IPSec.

I don’t have a clear understand of how Azure’s route-based and policy-based VGW options have proceeded over time, but it seems that there was once the option upon creation of a VGW to choose between the two methods and some PowerShell tomfoolery was required to configure a policy-based tunnel on a VGW that had been created as route-based.

The current state of affairs is that when creating a VGW, you no longer choose its traffic selector type. A newly spawned VGW will support route-based, policy-based, or route-based with simulated policy-based traffic selectors, and that last option is a button instead of PowerShell. Azure restricts what Internet Key Exchange (IKE) version you are able to configure based upon the VPN selected method. Route-based requires IKEv2 and policy-based requires IKEv1. If IKEv2 is used, then route-based in Azure must be selected and ASA must use a VTI.

## Conclusion

That wraps up Part 1 where I covered the foundational background needed to understand IPSEC VPNs. In Part 2, I’ll actually start configuring stuff.