---
title: "Palo Alto Firewalls in AWS (Part 2)"
draft: true
date: "2025-08-11"
series: ["Palo Alto Firewalls"]
series_order: 1
categories: ["Networking", "Cloud", "Security"]
tags: ["Palo Alto", "AWS"]
---

## Overview

https://weberblog.net/route-vs-policy-based-vpn-tunnels/

### Policy-based VPNs

A policy-based VPN does NOT use the routing table but a special additional policy to decide whether IP traffic is sent through a VPN tunnel or not. This policy is similar to policy-based routing which takes precedence over the normal routing table. Hence there are NO routing statements about the remote networks within the routing table.

* The IPSEC tunnel is invoked during policy lookup for traffic matching the interesting traffic 
* There are no tunnel interfaces. The remote end of the interesting traffic has a route pointed out through the default gateway 
* As there are no tunnel interfaces, we cannot have routing over VPNs
* The polices/access-lists configured for the interesting traffic serve as the proxy-IDs for the tunnels
* Firewalls that support policy-based VPNs: Juniper SRX, Juniper Netscreen, ASA, and Checkpoint

## Route-based VPNs

As the name implies a route-based VPN is a connection in which a routing table entry decides whether to route specific IP connections (based on its destination address) into a VPN tunnel or not. This routing statement is placed in the routing table of the firewall/router such as any other static/dynamic/connected routes.

A route-based VPN does NOT need specific phase 2 selectors/proxy-IDs. They can be ignored since every firewall sets them to ::/0 respectively 0.0.0.0/0 if not specified otherwise. This single VPN tunnel will have only one phase 1 (IKE) tunnel / security association and again only one single phase 2 (IPsec) tunnel / SA.

* The IPSec tunnel is invoked during route lookup for the remote end of the proxy-IDs.
* The remote end of the interesting traffic has a route pointing out through the tunnel interface.
* Support routing over VPNs.
* Proxy-IDs are configured as part of the VPN setup.
* Firewalls that support route-based Firewalls: Palo Alto Firewalls, Juniper SRX, Juniper Netscreen, and Checkpoint.

## Proxy ID's

A route based VPN doesn't care about Phase 2 subnets (typically set to 0.0.0.0/0 in the SA), but a policy based VPN does.

Most vendors that are route based (like palo alto) allow you to add "Proxy ID's" to emulate a policy based VPN, so you can build a tunnel to a policy based endpoint and have the phase 2 SA's match.

Palo Alto Network firewalls do not support policy-based VPNs. The policy-based VPNs have specific security rules/policies or access-lists (source addresses, destination addresses and ports) configured for permitting the interesting traffic through IPSec tunnels. These rules are referenced during the quick mode/IPSec phase 2, and are exchanged in the 1st or the 2nd messages as the proxy-ids. If the Palo Alto Firewall is not configured with the proxy-id settings, the ikemgr daemon sets the proxy-id with the default values of source ip: 0.0.0.0/0, destination ip: 0.0.0.0/0 and application:any, and these are exchanged with the peer during the 1st or the 2nd message of the quick mode. A successful phase 2 negotiation requires not only that the security proposals match, but also the proxy-ids on either peer, be a mirror image of each other.

So it is mandatory to configure the proxy-IDs whenever you establish a tunnel between the Palo Alto Network firewall and the firewalls configured for policy-based VPNs.

## AWS Make Ready
