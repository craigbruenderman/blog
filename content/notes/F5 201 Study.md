---
title: "F5 201 Study Notes"
author: Craig Bruenderman
geometry: margin=2cm
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["F5", "Certification"]
---

## Objective 1.01

### Packet processing order

1. Existing connection in connection table
1. Packet filter rule
1. Virtual server
1. SNAT
1. NAT
1. Self-IP
1. Drop

### Virtual Server Processing Order

1. address:port
1. address:*
1. network:port
1. network:*
1. *:port
1. star:star

## Objective 1.02

* Enabled - the virtual server is up and available for traffic (monitors are succeeding) and is represented by a green circle icon.

* Offline - the resource for the virtual server is not available (likely a failing monitor) and is represented by a red diamond icon.

* Currently Unavilable - the virtual server or all of it’s resources have reached a restricting connection limit that has been set by the administrator and the virtual server currently has no further capacity for traffic until the current connections fall below the connection limit settings. A yellow triangle icon represents the Currently Unavailable status.

* Unknown - there is not any monitors set for the resources of the virtual server, so there is no status to show and is represented by a blue square icon. This status does not mean that the virtual server will not respond to traffic. A virtual server with an Unknown status will take in traffic and send it on to the resources even if they are not online.

* Disabled - the administrator has marked the virtual server down so that it will not process traffic. The status icon will be a shape that represents the current monitor status of the virtual server but will always be colored black. Examples of this status icon would be; if the virtual server has succeeding monitors but is disabled the icon would be a black circle, or if the virtual server has failing monitors but is disabled the icon would be a black diamond or if the virtual server has no monitors but is disabled the icon would be a black square.

		show ltm virtual

## Objective 1.03

## Objective 1.04

## Objective 1.05

## Objective 1.06

## Objective 4.01

### Management Options

* Management port
* Console
* Self-IP

		list sys management route

## Objective 4.02

* Mirroring and NFO use TCP 1028

		tmsh list net self-allow

## Objective 4.03

* Network -> Packet Filters
* Uses `tcpdump` syntax

## Objective 5.01

* QKView gets uploaded to ihealth.f5.com
* System -> Support -> QKView enable and start
* `qkview` dumps to `/var/tmp`, which is volatile between reboots
* Severity 1 - 1-hour response
* Severity 2 - 1-hour response
* Severity 3 - 4 business hour response
* Severity 4 - NBD

## Objective 6.01

* Explain the icons on the map

## Objective 6.04



## Objective

## Objective

## Objective

## Objective

## Objective

