---
title: "JNCIA-Junos"
author: Craig Bruenderman
geometry: margin=2cm
date: 2020-01-01
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["Certification", "Juniper"]
---

## Network Terminology

Collision Domain
: A continuous electrically conductive wire

* When a device transmits, the entire wires voltage increases
* A hub connects multiple wires
* A switch separates collision domains
* Duplex dictates whether a device may transmit and receive at the same time
* CSMA-CD detects if multiple devices transmit at the same time on a collsion domain

## Network Components

1. Physical Layer PDU is a bit
2. Data Link Layer PDU is a frame
3. Network Layer PDU is a packet
4. Transport Layer PDU is a segment or datagram
5. Session Layer
6. Presentation Layer
7. Application Layer

## Network Topologies

* Leaf-Spine Topology
    * Two-tier where leaves attach hosts and spines provide leaf aggregation
    * All leaves connect to all spines
    * Spines are not interconnected
    * Common in data centers

### LAN

* Collapsed Core
    * Typical of small business
    * Resilient and relatively scalable
    * Core and agg/dist combined
        * L2/L3 boundary here

* 3-Tier
    * Higher scale
    * Core, agg/dist, access

### WAN

* Hub and spoke
    * Pros
        * Fewer linnks
        * Low cost
        * Simple management
    * Cons
        * Bottleneck at hub
        * Possible SPOF at hub

* Physical Full Mesh
    * Pros
        * Maximum redundancy
    * Cons
        * Difficult management
        * Most number of links
        * Highest cost

* Partial Mesh
    * Pros
        * Fewer links than full mesh
        * More redundant than hub and spoke
        * Cheaper than full mesh
    * Cons
        * Management can be complex
        * Not all sites have redundancy

## Physical Cabling

* Shared media ethernet networks are broadcast media
* Serial connections are point to point

### Copper

### Fiber

* APC (green) connector is angle polished
* UPC (blue) is flat/dome polished

## Layer 2 Addressing

* Media Access Control Address (MAC)
* MACs must be unique within a broadcast domain
* MACs are 48-bits, written as 12 hex chars
    * 12 x 4-bit chars = 48-bits
* Switches main function is to learn what port each MAC lives on
* Switches look at SRC MAC of incoming frames to build CAM table
* Switches look at DST MAC, and either forward the frame to the CAM known port, or flood it to all ports, except the one it arrived on, as an unknown unicast
* Typical MAC age-out timer is 300s (5 minutes)

## IPv4 Addressing

## IPv6 Addressing

* 128-bit addresses
    * 8 hextets, separated by colons
    * 16-bits per hextet
* Subnet mask works just like IPv4, only longer
* Address compression rules
    * Remove all preceeding 0's in a hextet
    * Replace contiguous hextets of 0's with double colon
        * Can only do this once per address, since otherwise you wouldn't know which hextet had which number of 0's
* Most common prefix length is /64
    * /64 is even recommended on point-to-point links due to EUI-64 compatibility

### IPv6 Address Types

* Global Unicast
    * 2000::/3 - analogous to IPv4 public space
* Unique Local
    * fc00::/7 - analogous to RFC1918 space
* Link Local
    * fe80::/64 - not routable
* Anycast
    * Same as global unicast, but belonging to multiple nodes
* Multicast
    * ff00::/8
* Modified EUI-64
    * Process used to self assign a globally unique IPv6 address

### Multicast

* ff02::1 - All IPv6 devices (broadcast)
* ff02::2 - All IPv6 routers
* ff02::5 - All OSPFv3 routers
* ff02::a - All IPv6 EIGRP routers

## Longest Match Routing

* If multiple routing entries include the destination IP, the longest match (most specific) route is used

### Prefix Length

* The length of a subnet mask is called the prefix length
* THe longer the prefix, the more host bits in the address

### Routing Table Contents

* Prefix
* Protocol
* Admin Distance
* Next-hop, including outgoing interface
* Age

## Class of Service

### Layer-2 traffic differentiation

* During times of congestion, we need to prioritize traffic
* Otherwise, traffic is dropped unpredictably
* 802.1q headers provides User Priority
    * 3-bit CoS field, decimal 0-7
    * The values can be mapped/used any way we want, but there is a typical convention where higher decimal value is higher priority (lower drop probability)

### Layer-3 QoS
* Uses 6-bit IP header field DSCP

## TCP vs. UDP

### TCP (Transmission Control Protocol)

* TCP is connection-oriented
* Header is 20-60 bytes
* Starts with a random initial sequence number
* Acknowledgement numner is set to the last sequence number that was received
* Include source and dest IP and port
* Window size indicates the maximum number of bytes that may be sent without an acknowledgement
* Options and padding field is up to 40 bytes

#### TCP Flags

* URG (Urgent) - Process packet before any non-urgent packets
* ACK (Acknowledge) - Ack receipt of a sequence number
* PSH (Push) - Packet processed immediately, not buffered
* RST (Reset) - Resets connection
* SYN (Synchronize) - Request to open a connection
* FIN (Finished) - Indicated sender has no more data to send and would like to close

### UDP (User Datagram Protocol)

* UDP is connectionless
* Protocol Dat Unit is datagram
* 8-byte header
* Includes source and destination IP and port

# Junos OS Fundamentals

## Juniper Software Architecture

* FreeBSD
    * Separate processes allow for segregation of tasks
* Same source code base is used in all Juniper hardware
    * Some has additonal features that run as separate processes
    * This make portfolio largely similar to operate
* Process "daemons" are the name of a service or process that runs in the background

### Common Daemons

* rpd - Routing Protocol Daemon
    * Functions include routing table updates, implementation of routing policy, and handling of routing messages
* mgd - Management Daemon
    * Serves as the broker between the external entities interacting with the device and the internal lrocesses
    * Manages the setup of the device when in configuration mode, and the collection of information from other processes when in operational mode
* dcd - Device Control Daemon
    * Responsible for configuring interfaces based on current configuration and available hardware

## Control and Forwarding Plane

* Control Plane is made of up the Routing Engine (RE) which is responsible for rpd, handling chassis components, system management, user access, etc
    * The RE manages the RT and from it dervices the Forwarding Table (FT)
* Forwarding Plane is made up of the Packet Forwarding Engine (PFE)
    * PFE uses the FT to forward transit traffic through the device
    * Any traffic which doesn't match to the FT or requires further processing will be forwarded to the RE

### Routing Engine

* RE runs Junos OS and consists of all processes and PCI platform which the OS runs on
* 6 main functions
    * Handling of routing protocol packets
    * Management interface
    * Configuration management
    * Accounting and alarms
    * Modular software
    * Scalability

### Packet Forwarding Engine

* PFE's primary job is to handle and forward transit traffic to its destination as fast as possible
* 3 components with ASICs
    * Switching control board
    * Physical Interface Card (PIC)
    * Flexible PIC Concentrator (FPC)

## Traffic Processing

### Transit Traffic

* Transit traffic is handled by the PFE
    * It's never seen by the RE
    * Traffic is not destined for the Juniper device itself
    * Its destination has an entry in the FT and is entirely handled by the PFE

### Exception Traffic

* Exception traffic needs to be processed by the RE
* This traffic is destined for the Juniper device itself
    * Management (SSH, Telnet, etc)
    * ICMP destined for the device
    * Routing protocol messages
    * Punted ICMP traffic
    * IP traffic with IP options set

# User Interfaces

## CLI Overview

### BSD Shell

* FreeBSD shell is only accessible from root
* Prompt looks like `root@:! #`
* When loggin in as root, you will be placed into the BSD shell
* No routing, switching, or security operations can be done here
* This is for interacting with the underlying Unix OS and the filesystem
* Unix commands like top, mkdir, mount, cat etc. available here

### Operational Mode

* Issue `cli` command to enter operational mode
* Prompt looks like `root>`
* This is the mode for monitoring, troubleshooting

### Configuration Mode

* Also called edit mode
* Issue `configuration` or `edit` command to enter configuration mode
* Prompt looks like `root#`

### Navigation

* Configuration hierarchy current position is displayed above the edit mode prompt
* Using `up` keyword will move up one level
* Using `top` keyword will return to the root of the hierarchy
* Configuration can be `set` from anywhere in the hierarchy, just depends on how much of the navigation must be specified
* `help apropos`
* ` help topic [topic]`

#### Filtering

* `match`
* `find`
* `display`
* `compare`

## Junos Configuration Concepts

* `show configuration` shows active config
  * Stored in `/config/juniper.conf.gz`
* `show | compare` shows what is in candidate config
* `commit check` checks syntax and sanity
* `commit confirmed` will automatically roll-back if you don't confirm it
  * Default is 10 minutes
  * Follow it up with `commit` to confirm
* `commit at` for scheduling
* `commit comment` lets you assign comment

## Working with Junos Configurations

### Configuration Load Sequence

* First looks for /config/juniper.conf.gz
* If not found, tries to load rescue config from the set path
* If not found, tries to load /config/juniper.conf.1.gz
  * `rollback 1` loads the above
* If not found, tries to load /etc/config/factory.conf

### Viewing Config

* `show configuration` hierarchical gives JSON-y view
* `show configuration | set display` gives procedural view that can be copy/pasted directly
* `show | compare` shows what is in candidate config in diff view

### Loading and Rollback

* `rollback [n]` loads a previous config to the candidate config
* `load [factory-default | merge | override | patch | replace | set | update]` loads a previous config to the candidate config

## J-Web Overview

* Must set root pass first
* Various devices have slightly different J-Web enable steps


# Configuration Basics

## Initial Configuration

* Hostname is set to Amnesiac by default
  * This is a hint a device is factory defaulted
* An unset root password another hint a device is factory defaulted
  * Junos forces a root password changed before committing any other config
* `set system root-authentication plain-text-password`
* `set system host-name <hostname>`
* `set system ntp server <pool.ntp.org>`
* `set system syslog host [host] any any` facility and severity
* `set snmp client-list ClientList1 10.0.0.0/24` source IP's allowed to poll
* `set snmp community <t3stcommunity>`
* `set system login user [username] authentication plain-text-password`
* `set system login user [username] class <class>`

## Authentication Config

### Login Classes

* RBAC default 4 login classes
  * Operator
  * Read-only
  * Super-user
  * Unauthorized

### User authentication methods

* Juniper supports 3
  * RADIUS
    * `set system radius-server [address]`
  * TACACS+
    * `set system tacplus-server [address]`
  * Local
* `set system authentication-order [password|radius|tacplus]`
  * This will always fall back to local on communication failure (timeout or no response)

## Juniper Device Interfaces

### Interface Categories

* 2 categories
  * Network - Physically connect to network and carry network traffic
  * Special = Can be virtual or physical and perform functions like routing, monitoring, and management
* 5 types of Interfaces
  * Permanent
  * Transient - Removable
  * Network
  * Services
  * Container
* Interface naming
  * interface-type-fpc/pic/port.unit

## Logging and Tracing

* Log files are stored in /var/log
* /var/log/messages stores system logs
  * `show log messages`

### Syslog Configuration

* Configured in [edit system syslog] hierarchy

### Traces

* Used for debugging
* Saved to a file in /var/log
* `traceoptions`
* `show log <filename>` to view them

## Advanced Configuration Functions

### Configuration groups

* Used to limit repeated commands in config files
* Inheritance can be excluded

### Archival

* Active config can be periodically backed up locally or off device
* File name is not customizable `<router_name>_YYYYMMDD_HHMMSS_juniper.conf.n.gz`
* Periodic interval can be immediate or specified
  * Transfer interval is between 15 and 2880 minutes (2 days)
  * Only will be transferred if something has changed
  * `transfer-on-commit` or `transfer-interval`
* `edit system archival`

### Rescue

* `request system configuration rescue save`

# Operation Monitoring and Maintenance

## Gather information and Monitoring

* `show interfaces [detail | description | terse]`
* `show route`
* `show chassis environment`
* `show chassis fpc pic-status`
* `monitor interface` shows live interface traffic stats
* `monitor traffic`
* `monitor start messages` real time display of messages log file
* `show configuration | display set`

## Troubleshooting

* `show interfaces <interface> statistics [detail]`

### Connecting to the device

* When SSH management is allowed, keys get created automatically
* `set system services ssh`

### Root Password Recovery

* Boot into single user mode
  * Reboot and issue `boot -s`
* Issue `recovery`

## Maintenance

* `request system snapshot media usb` takes config and OS backup
  * This will partition the USB drive
* `request system software add /tmp/<junos_filename>`
* `request system software add ftp://username:password@10.0.0.1/path/<junos_filename>`
* `request system reboot`

# Routing Fundamentals

## Routing Table vs. Forwarding Table

* RT is built by the control plane
* RT contains destination and path information
* inet.0 is the main IPv4 unicast table for local, direct, static and dynamically learned routes
* RT's
  * inet.1 is the multicast (PIM) table
  * inet.2
  * inet.3 is for IPv4 MPLS
  * inet6.0
  * mpls.0
  * Juniper_private
* FT contains forwarding information that the PFE uses to handle transit traffic
* `show pfe route ip` will show forwarding table on the PFE

## Traffic Forwarding Decisions

### Route Selection

* Juniper preference is the same as Cisco AD

| Source      | Default Preference |
| ----------- | ----------- |
| Directly Connected      | 0       |
| Static   | 5        |
| OSPF Internal   | 10        |
| IS-IS Level 1 Internal | 15 |
| IS-IS Level 2 Internal | 18 |
| Router Discovery | 55 |
| RIP | 100 |
| Aggregate | 130 |
| OSPF External | 150 |
| BGP | 170 |

## Tables and Instances

## Dynamic Routing

### Introduction

* Neighborship / Adjacency / Peering are the relationships between routers speaking a common dynamic protocol
* The overall purpose of dynamic routing protocols in for routers to automatically find paths to destination networks
* Each routing protocol operates differently with its own properties and advantages

### Distance Vector

* Unaware of the network as a whole, only their neighbors perspective.

# Routing Policy and Firewall Filters

## Introduction to Routing Policies

### Evaluation

* Organized into terms
* Terms are names and evaluated from top top bottom
* Term names are only used for network admins to describe them
* Terms are evaluated until a terminating action is reached
  * Accept, Reject, and Default are the actions
  * Evaluation continues until one of these is reached

```
policy-statement my_policy {
  term match-all-ipv4 {
    from family inet;
    then {
      next term;
      accept;
    }
  }
  term match-everything {
    then accept;
  }
  term match-nothing {
    from interface ge-0/0/0.0;
    then reject;
  }
}
```

## Configuring Routing Policies

### Match Criteria

* Where is the route from?
* Route-filters

### Default Policies

## Introduction to Firewall Filters

### Match Criteria

*
