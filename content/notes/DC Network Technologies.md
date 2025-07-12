---
title: "Data Center Network Technology Hotness"
author: Craig Bruenderman
date: 2015-01-27
geometry: margin=2cm
toc: true
output: pdf_document
categories: ["Networking"]
---

## Overall

* VXLAN, OTV, and LISP may share similar frame format, but they are complimentary and serve very different networking purposes
* Some overlap in functionality between the protocols
* Requirements / circumstances dictate when each is appropriate

## Challenge Areas

### Operational Model

* VLAN moves/adds/changes are laborious and error prone
* Coordination is required among the network admin, the cloud admin and the tenant admin to transport VLANs over existing switches

### Physical Topology Shortcomings

* VLANS today are too restrictive for virtual data centers in terms of physical constraints of distance and deployment
* It is desireable for broadcast domains to span within, and external to, the data center
* This is not possible over routed boundaries
* Various L2 extension techniques exist, but suffer various drawbacks

### MAC/VLAN Scale

* As we know, 802.1q only provides 12-bits for a total of 4096 VLANs
* This isn't normally a problem at the scales we deal in, but can be in mutli-tenant situations
* Many platforms support far fewer than this theoretical limit (e.g., Catalyst 3550 supports max 128 VLANs)

### Spanning Tree Shortcomings

# VSS

# VPC

# VXLAN

* VXLANs use Internet Protocol (both unicast and multicast) as the transport medium
* The ubiquity of IP networks and equipment allows the end to end reach of a VXLAN segment to be extended far beyond the typical reach of VLANs using 802.1Q today
* VXLAN is the basis of a scalable cloud network where lots of logical networks can be created instantly for complex and dynamic clouds
* Support for over 16M, courtesy of a 24 bit of logical network identifier
* VXLAN is not a vendor-specific technology (co-authored by many companies in both the virtualization and networking space), but it’s still an IETF draft (not a finalized standard) and product implementations by vendors will vary.
* VXLAN is intended for creating more logical networks in a cloud environment
* Locator ID Separation Protocol (LISP) goes a step further by providing IP address mobility between data centers with dynamic routing updates

## VTEPs

* VTEPs are intended to be at the edge of the network, typically connecting an access switch (virtual or physical) to an IP transport network
* It is expected that the VTEP functionality would be built into the access switch, but it is logically separate from the access switch
* In the case of a VXLAN enabled switch, the bridge domain would instead by associated with a VXLAN ID
* Each VTEP function has two interfaces
	* One is a bridge domain trunk port to the access switch, and the other is an IP interface to the IP network
* The VTEP behaves as in IP host to the IP networkand is configured with an IP address based on the subnet its IP interface is connected to
* The VTEP uses this IP interface to exchange IP packets carrying the encapsulated Ethernet frames with other VTEPs
* A VTEP also acts as an IP host by using the Internet Group Membership Protocol (IGMP) to join IP multicast groups
* In addition to a VXLAN ID to be carried over the IP interface between VTEPs, each VXLAN is associated with an IP multicast group
* The IP multicast group is used as communication bus between each VTEP to carry broadcast, multicast and unknown unicast frames to every VTEP participating in the VXLAN at a given moment in time
* The VTEP function also works the same way as a learning bridge, in that if it doesn’t know where a given destination MAC is, it floods the frame, but it performs this flooding function by sending the frame to the VXLAN’s associated multicast group
* Learning is similar, except instead of learning the source interface associated with a frame’s source MAC, it learns the encapsulating source IP address
* Once it has learned this MAC to remote IP association, frames can be encapsulated within a unicast IP packet directly to the destination VTEP

## Limitations

* The first implementations of VXLANs will be only in virtual access switches (the ones Virtual Machines connect to), so this means that only VMs can connect to VXLANs
* If a VM wants to talk with a physical device such as a physical server, layer 3 switch, router, physical network appliance, or even a VM running on a hypervisor that does not support a VXLAN enabled access switch, then it must use a VLAN
* Note that one downside to any encapsulation approach, whether it is based on UDP or GRE is that by having the hypervisor software add an encapsulation, today’s NICs and/or NIC drivers do not have a mechanism to be informed about the presence of the encapsulation for performing NIC hardware offloads.

## Comparison

* If one were to look carefully at the encapsulation format of VXLAN one might notice that it is actually a subset of the IPv4 OTV encapsulation in draft-hasmit-ovt-03, except the Overlay ID field is not used (and made reserved) and the well-known destination UDP port is not yet allocated by IANA (but will be different)
* If one were to look even closer, they would notice that OTV is actually a subset of the IPv4 LISP encapsulation, but carrying an Ethernet payload instead of an IP payload
* Since VXLAN is designed to be run within a single administrative domain (e.g. a datacenter), and not across the Internet, it is free to use Any Source Multicast (ASM (*,G) forwarding) to flood unknown unicasts
* Since a VXLAN VTEP may be running in every host in a datacenter, it must scale to numbers far beyond what IS-IS was designed to scale to

## Motives

* Why did VXLANs use a MAC-in-UDP encapsulation instead of MAC-in-GRE?  The easy answer is to say, for the same reasons OTV and LISP use UDP instead of GRE.  The reality is that the vast majority (if not all) switches and routers do not parse deeply into GRE packets for applying policies related to load distribution (Port Channel and ECMP load spreading) and security (ACLs).
* If one of today’s switches were to try to distribute GRE flows between two VTEPs that used a GRE encapsulation, all the traffic would be polarized to use only one link within these Port Channels.  Why?  Because the physical switches only see two IP endpoints communicating, and cannot parse the GRE header to identify the individual flows from each VM.  Fortunately, these same switches all support parsing of UDP all the way to the UDP source and destination port numbers.  By configuring the switches to use the hash of source IP/dest IP/L4 protocol/source L4 port/dest L4 port (typically referred to as a 5-tuple), they can spread each UDP flow out to a different link of a Port Channel or ECMP route.  While VXLAN does use a well-known destination UDP port, the source UDP port can be any value.  A smart VTEP can spread the all the VMs 5-tuple flows over many source UDP ports.  This allows the intermediate switches to spread the multiple flows (even between the same two VMs!) out over all the available links in the physical network.  This is an important feature for data center network design.  Note that this does not just apply to layer 2 switches, since VXLAN traffic is IP and can cross routers as well, it applies to ECMP IP routing in the core as well.

# OTV

* OTV uses similar frame format as VXLAN, but is squarely intended as a data center interconnect L2 over L3 extension technology
* OTV has simpler deployment requirements than VXLAN since it does not mandate multicast-enabled transport network
* OTV does not create more layer 2 network segments, instead it extends the existing ones over IP

# LISP

* Locator/ID Separation Protocol (LISP) is a technology that allows end systems to keep their IP address (ID) even as they move to a different subnet within the Internet (Location).  It breaks the ID/Location dependency that exists in the Internet today by creating dynamic tunnels between routers (Ingress and Egress Tunnel Routers).  Ingress Tunnel Routers (ITRs) tunnel packets to Egress Tunnel Routers (ETRs) by looking up the mapping of an end system’s IP address (ID) to its adjacent ETR IP address (Locator) in the LISP mapping system.

* LISP provides true end system mobility while maintaining shortest path routing of packets to the end system.  With traditional IP routing, an end station’s IP address must match the subnet it is connected to.  While VXLAN can extend a layer 2 segment (and therefore the subnet it is congruent with), across hosts which are physically connected to different subnets, when a VM on a particular host needs to communicate out through a physical router via a VLAN, the VMs IP address must match the subnet of that VLAN -- unless the router supports LISP.

* If a VXLAN is extended across a router boundary, and the IP Gateway for the VXLAN’s congruent subnet is a VM on the other side of the router, this means traffic will flow from the originating VMs server, across the IP network to the IP Gateway VM residing on another host, and then back up into the physical IP network via a VLAN.  This phenomenon is sometime referred to as “traffic tromboning” (alluding to the curved shape of a trombone).  Thus, while VXLANs support VMs moving across hosts connected to different layer 2 domains (and therefore subnets), it doesn’t provide the direct path routing of traffic that LISP does.

* MAC-in-MAC

* VMware has an existing proprietary equivalent of VXLAN which is deployed today with vCloud Director, called vCloud Director Network Isolation (vCDNI).  vCDNI uses a MAC-in-MAC encapsulation.  Cisco and VMware, along with others in the hypervisor and networking industry have worked together on a common industry standard to replace vCDNI -- namely VXLAN.  VXLAN has been designed to overcome the shortcomings of the vCDNI MAC-in-MAC encapsulation -- namely load distribution, and limited span of a layer 2 segment.

* The first one is the same issue that GRE has with load distribution across layer 2 switch Port Channels (and ECMP for FabricPath). The second is that because the outer encapsulation is a layer 2 frame (not an IP packet), all network nodes (hypervisors in the case vCDNI), MUST be connected to the same VLAN.  This limits the flexibility in placing VMs within your datacenter if you have any routers interconnecting your server pods unless you use a layer 2 extension technology such as OTV to do it.

# FabricPath

# TRILL

# NSX

* NSX focuses on only the VM-to-VM traffic pattern, with no ability to move traffic between VMs on different physical devices
* With or without NSX, a physical network will be required to actually move packets between devices
* NSX provides no management or visibility into the physical network that the NSX application utilizes, you'll want to ensure that any network chosen for NSX provides native automation for provisioning and network change, as well as advanced visibility and telemetry tools for troubleshooting and day-two operations
* VMware-only NSX for vSphere, or the multi-hypervisor version, VMware NSX-MH
* The two products are not compatible, and the features vary greatly between the two
* NSX-MH is being phased out

# ACI

* ACI can tie both physical and virtual environments together and treat them equally from a connectivity, security, user experience, and auditing perspective.

## Entry Point

* Typically includes two low-cost, high-performance spine switches (32 ports of line-rate 40G each), which provide connectivity between leaf switches for interconnectivity, as well as other ACI functions
* It also includes two leaf switches (48 ports of line-rate 1/10G with 12 port of 40G uplink) for connectivity of servers, L4-7 services appliances, external network, WAN, etc.
* Last, the Application Policy Infrastructure Controller (APIC) cluster, and all required licensing
* Current pricing on an ACI starter kit is below $125,000 average street price
* ACI alleviates the need for gateway servers by providing gateway, bridging, and routing functions for untagged, VLAN, VxLAN, and NVGRE on any port