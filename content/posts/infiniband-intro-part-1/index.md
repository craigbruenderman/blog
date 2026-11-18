---
title: "Infiniband Intro (Part 1)"
draft: true
date: "2025-08-19"
series: ["Infiniband"]
series_order: 1
categories: ["Networking", "Cloud"]
tags: ["Infiniband", "Nvidia"]
---

## Overview

## History

* 2002
* SDR
* QDR
* FDR - 56 Gbps
* EDR - 100 Gbps
* HDR - 200 Gbps
* NDR - 400 Gbps
* XDR - 800 Gbps
* GDR - 1.6Tbps

## Infiniband Layers

* Upper Layer
* Transport Layer
* Network Layer
* Link Layer

## Physical Layer

* Host Channel Adapter (HCA)
  * Equivalent of an Ethernet NIC
  * Performs CPU offload
  * Single or multi-port options
  * ConectX by Nvidia is most common
    * These are unified, supporting both Ethernet and Infiniband

## Control Plane

#### GUID

* 64-bit addresses
* System GUID
* Node GUID
* Port GUID

#### LID

* 16-bit address assigned to each Infiniband port

### Subnet Manager

* Brain of the fabric that discovers, configures, and maintains the network
  * Assigns LIDS
  * Discovers topology
  * Programs routing tables
  * Ensures path consistency
  * Manages partitions (P_Key)
  * Handles link state changes
  * Optimizes performance
* One active Master Subnet Manager, with others configured in Standby Mode
* Subnet Manager must exist inside the fabric
  * Managed switches can run SM inside
  * This is the most common approach
* SM can also be run on a server with an HCA
  * OpenSM is typical for labs or small clusters
* Separate dedicated servers can also run
  * This is the preferred approach in large scale HPC & AI clusters

## Random

* Infiniband to Ethernet Gateway
* Typical Infiniband applications
  * IPoIP
  * NVMe over Fabrics
  * RDMA
* Nvidia OFED (Open Fabric Enterprise Distribution) contains driver packages to run IB
* Port Local ID
* GUID
* NCCL

## Infiniband Compared to Ethernet & TCP/IP

## Why Choose Infiniband?

## Infiniband Connectivity

## Subnet Manager

## Infiniband Addressing
