---
title: "ACI Fundamentals"
author: Craig Bruenderman
geometry: margin=2cm
date: 2020-01-01
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["ACI", "Cisco"]
---

## Cisco ACI Fabric Infrastructure and Basic Concepts

### What is ACI?

* Key benefits of Cisco ACI:
    * Automation of IT workflows and application deployment agility
    * Open APIs and a programmable SDN fabric, with 65-plus ecosystem partners
    * Security through allow lists, policy enforcement, microsegmentation, and analytics
    * Workload mobility at scale for physical and virtual load
* Spine switches:
    * Represent the backbone of the ACI fabric
    * Connected to leaf switches
* Leaf switches
    * Represent connection point for end devices, including APIC
    * Connected to spine switches
* APICs
    * Unified point of policy enforcement, health monitoring, and management for the Cisco ACI fabric
    * Not involved in data plane forwarding

### ACI Topology and Hardware

#### Spine-Leaf Topology Benefits

* Simple and consistent topology
* Scalability for connectivity and bandwidth
* Symmetry for optimization of forwarding behavior
* Least-cost design for high bandwidth 

#### IS-IS Fabric Infrastructure Routing

The fabric applies a densely tuned Intermediate System-to-Intermediate System (IS-IS) environment utilizing Level 1 connections within the topology for advertising loopback addresses. Loopback addresses are the Virtual Extensible LAN (VXLAN) Tunnel Endpoints (TEPs) that are used in the integrated overlay and advertised to all other nodes in the fabric for overlay tunnel use.

* IS-IS provides IP reachability among TEP addresses
* Automatically deployed, no user intervention is required
* No IS-IS knowledge is required

#### Endpoint Forwarding Across Leaf Switches

When a packet is sent from one leaf to another, an end host (called an endpoint in ACI) location is identified by a TEP IP of each node. User data traffic is encapsulated with VXLAN header when being forwarded to another leaf. The forwarding across switch nodes is performed based on the TEP IP in the VXLAN encapsulation. In case the ingress leaf is not aware of the destination endpoint location (TEP), ACI has a distributed database called Council of Oracles Protocol (COOP) on each spine that knows all the mapping of endpoint and TEP.

### ACI Object Model

* Security Group (Security/Application Tenant Policies)
    * Defines how ACI fabric should allow network communication between two devices. Example policies are endpoint groups (EPGs), contracts, and so on.
* Overlay Network (Network Tenant Policies)
    * Defines the overlay network that goes over the ACI fabric. These policies will be the logical topology of user networks and can be defined without worrying about the underlay. Example policies are VRF, bridge domain (BD), and so on.
* Fabric Access (Access Policies)
    * Defines how end hosts or external network devices are connected to ACI fabric by configuring port-channel, vPC, and other interface-level configurations.
* Underlay Network (Fabric Policies)
    * Defines the VXLAN underlay built by ACI fabric along with infra Multiprotocol Border Gateway Protocol (MP-BGP) and other protocols to manage ACI switches such as NTP.

### Faults, Event Record, and Audi

* Based on object status or changes, the Cisco APIC generates the following objects for logging purpose:
    * Faults: for issues in the fabric (such as an issue in configuration or a fan failure on a node)
    * Events: for events in the fabric (similar to `show logging log` in standalone NX-OS)
    * Audit Logs: for configuration changes in the fabric (similar to `show accounting log` in standalone NX-OS)

* The following are the objects for events and audit logs:
    * eventRecord: an object for events
    * aaaModLR: an object for configuration changes (audit logs)
    * aaaSessionLR: an object for logins and logouts (audit logs)

### ACI Fabric Discovery

### ACI Access Policies

#### Access policies are grouped into the following categories:

* Pools: Specify VLAN and multicast address pools.
* Interface profiles: Specify which access interfaces to configure and the interface configuration policy.
* Switch profiles: Specify which switches to configure and the switch configuration policy.
* Module profiles: Specify which leaf module to configure. But as of release 4.1, there is no leaf model that has more than one module. Hence this profile is typically never used.
* Global policies: Enable the configuration of DHCP, QoS, and Attachable Access Entity Profile (AAEP).
* Physical and external domains: Define a domain that bundles a set of interfaces (AAEP) and encapsulations (VLAN pool) to allow other components.
* Monitoring and troubleshooting policies: Specify what to monitor, the thresholds, how to handle faults and logs, and how to perform diagnostics related to external facing interfaces.

### Create Access Policies & vPC

## Cisco ACI Policy Model Logical Constructs

### ACI Logical Constructs

#### Tenant Types

* Common: A special tenant that provides services that are common to other tenants in the Cisco ACI fabric. Global reuse is a core principle in the common tenant. Examples of common services include Domain Name System (DNS), DHCP, and Active Directory.
* Infra: The infrastructure tenant that is used for all internal fabric communications, such as tunnels and policy deployment, including switch-to-switch (leaf, spine, Cisco Application Virtual Switch [AVS], or Cisco Application Virtual Edge [AVE]) and switch-to-APIC. The infra-tenant does not get exposed to other tenants. It has its own VRF and bridge domains. Fabric discovery, image management, and DHCP for fabric functions are all handled within the infra-tenant. Users do not need to manually configure components in this tenant with a few exceptions such as when configuring ACI multi-pod infra.
* Mgmt: The management tenant is provided by the system but can be configured by the fabric administrator. It contains policies that govern the operation of fabric management functions used for in-band and out-of-band configuration of fabric nodes. The management tenant contains a private out-of-bound address space for the APIC/fabric internal communications that is outside the fabric data path that provides access through the management port of the switches. The management tenant enables discovery and automation of communications with virtual machine controllers.
* User tenants: are defined by the administrator according to the needs of users. They contain policies that govern the operation of resources such as applications, databases, web servers, network-attached storage, virtual machines, and so on.

### Tenant

At the top level, the Cisco APIC policy model is built on a series of one or more tenants. A tenant is a logical container for application policies that enable an administrator to exercise domain-based access control. The fabric can contain multiple tenants. A tenant represents a unit of isolation from a policy perspective, but it does not represent a VRF. Tenants can represent a customer in a service provider setting, an organization or domain in an enterprise setting, or just a convenient grouping of policies.

* Can represent a customer, business unit, or group.
* Provides a separate profile space.
* Tenants only see inside their space.
* Shared services can be defined between tenants.

### VRFs

### Bridge Domains

#### Characteristics of bridge domains are

* Layer 2 Forwarding domain
* Provide a default gateway and subnet for endpoints
* Belongs to one VRF
* One or more bridge domains per tenant
* One or more bridge domains per VRF
* Can consist of multiple subnets

### Endpoint Groups

* EPGs are used to create logical groupings of hosts or servers (endpoints) that perform similar functions within the fabric.
* A new concept in ACI, that a typical network infrastructure did not have by default. ACI defines multiple endpoint groups (EPG) within a Layer 2 domain (BD) for security isolation purposes on top of Layer 2 network separation. In a traditional network device, Layer 2 network separation is the smallest segmentation, that is achieved via VLAN ID. However, since ACI Layer 2 domain (BD) is not directly tied to a VLAN ID, ACI can provide one more layer of segmentation using VLAN ID that is smaller than Layer 2 domain (EPG). Hence, in ACI, EPG is a security segmentation smaller than Layer 2 domain and VLAN ID is a parameter for security separation instead of Layer 2 network separation.
* Since EPG is to security separation, no endpoints can talk to each other across EPGs unless it’s explicitly allowed, which is performed by a policy called contract. Any endpoints within a same EPG can talk to each other because they are not segmented from each other.

#### The EPG features can be summarized in this way:

* A group to provide more granular segmentation than Layer 2 network separation
* A group of endpoints with a similar security requirement
* Unrestricted intra-EPG communication
* Inter-EPG communication controlled by contracts
* An EPG belongs to a bridge domain
* A bridge domain may contain multiple EPGs

#### Application Profiles

* Application profiles are groups of EPGs. Each application profile created can have a unique monitoring policy and QoS policy applied.

### ACI Management

#### Out of Band

# Cisco ACI Basic Packet Forwarding and External Network Connectivity

## Cisco ACI Basic Packet Forwarding

### Endpoint Learning

* In a traditional network, three tables are used to maintain the network addresses of external devices:
    * A MAC address table for Layer 2 Forwarding
    * A Routing Information Base (RIB) for Layer 3 forwarding
    * An ARP table for the combination of IP addresses and MAC addresses
* Cisco ACI learns that information in a different way than in a traditional network
* Cisco ACI replaced the MAC address table and ARP table with a single table called the endpoint table
 * ACI learns MAC and IP addresses in hardware by looking at the packet source MAC address and source IP address in the data plane instead of relying on ARP to obtain a next-hop MAC address for IP addresses
 * This approach reduces the amount of resources needed to process and generate ARP traffic
 * It also allows detection of IP address and MAC address movement without the need to wait for GARP as long as some traffic is sent from the new host
 * Although Cisco ACI uses the endpoint table instead of the MAC address and ARP tables, it still uses the RIB and the ARP table for L3Out
* Forwarding table lookup order:
    1. Endpoint table (show endpoint)
    2. RIB (show ip route)
* A leaf switch has two types of endpoints:
    * Local endpoints stored on the endpoint table on each leaf
    * Remote endpoints stored on the endpoint table on each leaf only when a conversation to the endpoint is happening (conversational learning)
* Although both local and remote endpoints are learned from the data plane, remote endpoints are merely a cache, local to each leaf
* Local endpoints are the main source of endpoint information for the entire Cisco ACI fabric
* Each leaf is responsible for reporting its local endpoints to the Council Of Oracle Protocol, which is known as COOP, database, located on each spine switch
* Spine switch stores these endpoints in the COOP database and synchronizes with other spine switches
* Because this database is accessible, each leaf does not need to know about all the remote endpoints to forward packets to the remote leaf endpoints
  * When leaf switch does not know the destination endpoint, leaf can forward packet to spine switch to let spine switch decide where to send
  * This forwarding behavior is called spine proxy

### Basic Bridge Domain Config

* The basic bridge domain configuration options that should be considered when configuring a bridge domain behavior are
    * Whether to use hardware proxy or flooding mode for Layer 2 Unknown Unicast packets
    * Whether to enable or disable Address Resolution Protocol (ARP) flooding
    * Whether to enable or disable unicast routing
    * Whether or not to define one or more subnets under the bridge domain

* Hardware proxy for Layer 2 unknown unicast traffic is the default option. If the destination MAC is not in the ingress leaf endpoint table, the packet is sent to the spine proxy. This forwarding behavior uses the COOP database on spine switches to forward unknown unicast traffic to the destination leaf without relying on flood-and-learn behavior, as long as the MAC address is known to the COOP database on spine switch.
* With Layer 2 unknown unicast flooding, that is, if hardware proxy is not selected, the leaf endpoint table and spine COOP database are still populated with the MAC-to-VTEP information. However, the forwarding does not use the COOP database on spine switches. Layer 2 unknown unicast packets are flooded within the bridge domain.
* If ARP flooding is enabled, ARP traffic will be flooded within the bridge domain as per regular ARP handling in traditional networks. If this option is disabled, the ingress leaf uses unicast to send the ARP traffic to the destination leaf or to spine-proxy (detailed scenarios are explained later in this content). Note that these options apply only if unicast routing is enabled on the bridge domain. If unicast routing is disabled, ARP traffic is always flooded within the bridge domain.

## External Network Connectivity

### Cisco ACI External Connectivity Options

* In Cisco ACI, any end hosts or network devices that are learned as an endpoint via normal EPG is considered “inside” ACI fabric. ACI provides those endpoints with an exit to other network domains which is referred to as “outside” or External Connections.
* This outside network could be another simple Layer 2 network with many non-ACI switches, called Layer 2 External Network Connectivity, which can be achieved by Layer 2 Out (L2Out) or EPG/VLAN Extension.
* Or it could be a Layer 3 network, called L3Out where ACI needs to learn about it via the routing protocol, or static route
* L3Out to external networks have these characteristics:
  * Link to network that contains multiple subnets
  * Provide reachability via OSPF, BGP, EIGRP, or static routes
* L2Out or EPG/VLAN Extension to external networks have these characteristics:
  * Extend the Layer 2 domain (bridge domain) outside of the Cisco ACI fabric
  * Support VLAN for tagging

### External Layer 2 Network Connectivity

* There are two common ways of extending a Layer 2 domain outside the Cisco ACI fabric:
  * Extend EPG via a static path binding
  * Extend a bridge domain via a dedicated component called L2Out in the APIC GUI

## VMM Integrations

### VMware vCenter VDS Integration

#### Resolution Immediacy in VMM

Pre-provision: VLAN will be deployed on all leaf interfaces under the AAEP associated to the VMM domain regardless of VM controller of hypervisor status. This option should be used for critical services that require VLANs for them to be deployed all the time instead of dynamically only when needed. This is because this option has higher chance to hit VLAN port number limitation on a leaf since the VLAN is deployed on all interfaces in the AAEP. If AAEP is designed so that it only has interfaces connected to ACI-managed VDS, this option could be used for all EPG association. An example is vCenter and ESXi communication.

Immediate: VLAN will be deployed on leaf interfaces only when hypervisors are detected through LLDP or CDP. This information has to be bi-direction. It means APIC compares the LLDP/CDP information from leaf switch and a VM controller to ensure that the hypervisor and leaf switches are connected correctly. If APIC has only one-sided information such as only LLDP from leaf switch, the VLAN will not be deployed. In case there is an intermediate switch in between such as Cisco UCS Fabric Interconnect, both leaf interface and hypervisor uplink should show the same Cisco UCS Fabric Interconnect in its CDP or LLDP information.

On Demand: VLAN will be deployed on leaf interfaces only when hypervisors are detected as mentioned in Immediate mode and when at least one VM is associated to the corresponding port group. Both conditions need to be met for the VLAN to be deployed on leaf interfaces in On-Demand mode.

## Layer 4 through Layer 7 Integrations

### Service Appliance Insertion Without ACI L4-L7 Service Graph

Cisco ACI technology provides the capability to insert Layer 4 through Layer 7 functions using an approach called a service graph (described below). In Cisco ACI, you also can insert service devices such as load balancers in the path without a service graph. To do so, you need to treat them as External Layer 2 Connectivity (EPG Extension) or External Layer 3 Connectivity (L3Out). The figure below shows an example logical topology where the client VM reaches to the VIP of the load balancer, and the load balance will perform source NAT (SNAT) to hide the client IP address from back-end servers prior to forward packets to them. Contracts here are used to allow only a specific type of traffic to reach to the load balancer, and eventually the back-end server.

### Service Appliance Insertion via ACI L4-L7 Service Graph

With the ACI L4-L7 Service Graph, some of the configurations can be automated. The biggest benefit of Service Graph is to allow users to focus on contracts between client VM and back-end servers instead of worrying about contracts between client VM and a load balancer, a load balancer, and back-end servers in the previous example. Although it simplifies and automates the contract security part of the configuration, users still need to carefully design the network topology to ensure that traffic will flow through a service node such as a load balancer. In case the traffic flow needs to be bent towards a service node against a routing table or endpoint table, the Service Graph Policy Based Redirection (PBR) feature needs to be used. The details of PBR will be covered in the advanced course and in this course, you will focus on a simple Service Graph.

### Service Graph Configuration

Concrete device: Represents a service device, physical or virtual.

Logical device: Represents a cluster of devices; defines logical interfaces.

Function node: A function node represents a function that is applied to the traffic, such as a firewall.

Terminal node: A terminal node enables input and output from the service graph.

Connector: A connector enables input and output from a node.

Connection: A connection determines how traffic is forwarded through the network.

All the configurations you saw until now need to be deployed in the end, which is what service graph rendering does. Service graph rendering means to apply the configuration on the actual ACI fabric. The rendering involves allocation of the necessary bridge domains, configuration of IP addresses on the firewall and load balancer interfaces, creation of the VLAN on these devices to create the path for the functions, and performance of all the work necessary to make sure that the path between EPGs is the path defined in the service graph.

The rendering involves allocation of the necessary VLANs between the L4-L7 device and the bridge domains. Cisco ACI creates so called internal EPGs to which the L4-L7 device connects, and it creates internal contracts to enable communication to and from the L4-L7 device.

### Service Graph PBR Introduction

The traffic in the Cisco ACI fabric is routed and bridged based on the destination IP and MAC addresses, which are the same as in traditional networks. When you use a service graph for service node insertion, the process is the same, and you must consider routing and bridging design as well.

Cisco ACI supports a policy-based routing (PBR) functionality, which can be used with a service graph to redirect traffic between security zones to inserted Layer 4 to Layer 7 devices, such as a firewall, load balancer, or Cisco Intrusion Prevention System (IPS), without the need for this device to be the default gateway for endpoints. It also removes the need to employ traditional service insertion models, such as VLAN and VRF stitching. The use of PBR simplifies service node insertion and removal, while the provisioning of service nodes can be performed using a service graph. In addition, you can use PBR to selectively send traffic to Layer 4 to Layer 7 devices based, for instance, on the protocol and specific Layer 4 port, you can insert a firewall in the transparent mode in a Layer 2 domain with almost no modification to existing routing and switching configurations, and so on.

The following figure shows two options for an insertion of a router firewall, which protects traffic flows between the external Layer 3 network domain and the web EPG. It illustrates the difference between a routing-based design (a classic VRF sandwich), which can be performed with service graph without PBR (or even with no service graph) and service graph with PBR.

The routing-based design requires multiple VRFs and Layer 3 outside (L3Out) connections, which are established between the fabric and the internal and external firewall interfaces. It represents a classic VRF sandwich configuration, where all traffic goes through the routed firewall. The firewall is a routed Layer 3 hop at the front end of the VRF2 instance, which encompasses the web subnet and the IP subnet of the firewall internal interface. The firewall outside the interface and the Layer 3 interface facing the WAN edge router are part of a separate VRF1 instance.

The use of PBR significantly simplifies the configuration, because the previously described VRF sandwich configuration is not required anymore. Hence, you can use single VRF, while the traffic is instead redirected to the service node based on the configured policy. Thus, you can implement selective traffic redirection, which is an advantage when compared to a service graph without PBR.

Starting from APIC Release 4.1, PBR can be used with Layer 4 to Layer 7 service devices operating in Layer 1 or Layer 2 mode. This functionality enables you to use L1/L2 PBR during insertion of service nodes, such as an inline IPS, transparent firewall, and so on.

The following figure shows two options for an insertion of a transparent firewall, which protects traffic flows between the external Layer 3 network and the web EPG. It depicts the difference between a classical bridge domain “sandwich” design, which can use a service graph without PBR to make the traffic go through the transparent firewall and service graph with PBR design.

As previously indicated, PBR mandates the usage of a service graph, which has consumer and provider EPGs, and a redirect policy that is programmed on the switches on which the consumer or provider EPG is located. The following figure depicts a use case that uses different PBR policy based on the source and destination EPG combination.