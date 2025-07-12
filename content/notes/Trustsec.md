---
title: "Trustsec Overview"
date: '2024-10-24'
author: Craig Bruenderman
geometry: margin=2cm
type: list
output: pdf_document
image: /images/cisco-trustec.jpg
---

## Terms

Security Group
: Used for grouping users, endpoints, and resources that should have a similar access control policy

Security Group Tag (SGT)
: Unique security group number that's assigned to a Security Group

Trustsec Capable Device
: Network access device that's hardware & software capable of understanding SGT's

Trustsec Seed Device
: Network access device that authenticates directly against ISE and acts both as the authenticator and supplicant for other network access devices

Protected Access Credential (PAC)
: Unique shared credential used to mutually authenticate client and server

Endpoint Authentication Control
: Devices authenticate to Trustsec via 802.1x, MAC, Webauth, etc

Security Group Access Control List (SGACL)
: These are used for access permissions based on SGTs, rather than IP's. This simplifies the security policy

Security Exchange Protocol (SXP)
: A protocol/service that's used to propagate IP to SGT bindings across network devices that don't support SGT's

Identity to port mapping
: Switch defining the identity on a port and using this identity to look up a particular SGT value from ISE

## Network Device Admission Control (NAC)
* In a Trustsec deployment, network devices are verified with credentials by the peer devices
  * 802.1x
  * EAP-FAST
  * Upon authentication and authorization, negotiates for IEEE 802.1ae encryption

## Environment Data Download
* A download from ISE to the network access device when it joins the trusted network
* When it does this, it downloads the following:
  * ISE RADIUS server list it can use for future RADIUS authentications and authorizations
  * Device SGT for the network access device itself
  * Expiry timeout for environmental download/refresh interval
