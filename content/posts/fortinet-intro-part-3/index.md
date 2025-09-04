---
title: "Part 3 - SD-WAN"
date: "2025-09-02"
draft: true
series: ["A look at Fortinet"]
series_order: 2
categories: ["Networking"]
tags: ["Fortinet"]
---

## Overview

In Part 3, I'll dive into the SD-WAN feature of Fortinet Fortigates.

## Fortinet SD-WAN Design principles

Fortinet lays out their Five-pillar approach, described in the [SD-WAN Architecture for MSSPs guide](https://docs.fortinet.com/document/fortigate/7.6.4/administration-guide/257828/sd-wan-components-and-design-principles).

### Underlay

The Underlay is the set of WAN links that will be used for the underlay network, such as broadband, MPLS, 4G/5G LTE connection, etc. The characteristics of each link (bandwidth, packet loss, latency, and jitter) and cost are the metrics used to determine which link to prefer, what type of traffic to send across the each link, and baselines for health-checks.

### Overlay

VPN overlays are needed when traffic must travel across multiple sites. These are usually site-to-site IPsec tunnels that interconnect branches, datacenters, and the cloud, forming a hub-and-spoke topology. The management and maintenance of the tunnels should be considered when determining the overlay network requirements. Manual tunnel configuration might be sufficient in a small environment, but could become unmanageable as the environment size increases. ADVPN can be used to help scale the solution; see ADVPN for more information.

### Routing

Traditional routing designs manipulate routes to steer traffic to different links. SD-WAN uses traditional routing to build the basic routing table to reach different destinations, but uses SD-WAN rules to steer traffic. This allows the steering to be based on criteria such as destination, internet service, application, route tag, and the health of the link. Routing in an SD-WAN solution is used to identify all possible routes across the underlays and overlays, which the FortiGate balances using ECMP.

In the most basic configuration, static gateways that are configured on an SD-WAN member interface automatically provide the basic routing needed for the FortiGate to balance traffic across the links. As the number of sites and destinations increases, manually maintaining routes to each destination becomes difficult. Using dynamic routing to advertise routes across overlay tunnels should be considered when you have many sites to interconnect.

### Security

Security involves defining policies for access control and applying the appropriate protection using the FortiGate's NGFW features. Efficiently grouping SD-WAN members into SD-WAN zones must also be considered. Typically, underlays provide direct internet access and overlays provide remote internet or network access. Grouping the underlays together into one zone, and the overlays into one or more zones could be an effective method.

## Fortinet SD-WAN Terminology

SD-WAN is fundamentally defined by the separation of control-plane and data-plane function, along with a management tool of some flavor. Some solutions like Arista Velocloud and Cisco SD-WAN have discrete components which directly map to each of these functional elements. Some, like Juniper SSE and Fortinet, combine the control and data-plane on the network elements themselves. Fortinet frames their solution in three layers:

* Management and orchestration
* Control, data plane, and security
* Network access

The control, data plane, and security layers can only be deployed on a FortiGate. The other two layers can help to scale and enhance the solution. Fortinet SD-WAN can be implemented and managed ad hoc on each participant Fortigate device, but it is really intended to use FortiManager and FortiAnalyzer for management and orchestration capabilities FortiSwitch and FortiAP are ancillary components to SD-WAN, but they round out the portfolio approach of an SD-Branch.

### SD-WAN Zones

SD-WAN is divided into zones. SD-WAN member interfaces are assigned to zones, and zones are used in policies as source and destination interfaces. You can define multiple zones to group SD-WAN interfaces together, allowing logical groupings for overlay and underlay interfaces. Routing can be configured per zone.

### SD-WAN Members

Also called interfaces, SD-WAN members are the ports and interfaces that are used to run traffic. At least one interface must be configured for SD-WAN to function.

### Performance SLAs

Also called health-checks, performance SLAs are used to monitor member interface link quality, and to detect link failures. When the SLA falls below a configured threshold, the route can be removed, and traffic can be steered to different links in the SD-WAN rule.

SLA health-checks use active or passive probing. Active probing requires manually defining the server to be probed, and generates consistent probing traffic. Passive probing uses active sessions that are passing through firewall policies used by the related SD-WAN interfaces to derive health measurements. It reduces the amount of configuration, and eliminates probing traffic. See Passive WAN health measurement for details.

### SD-WAN Rules

Also called services, SD-WAN rules control path selection. Specific traffic can be dynamically sent to the best link, or use a specific route. Rules control the strategy that the FortiGate uses when selecting the outbound traffic interface, the SLAs that are monitored when selecting the outgoing interface, and the criteria for selecting the traffic that adheres to the rule. When no SD-WAN rules match the traffic, the implicit rule applies.