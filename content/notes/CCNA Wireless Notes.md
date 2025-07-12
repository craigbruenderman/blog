---
title: "CCNA Wireless Study Notes"
author: Craig Bruenderman
date: 2016-09-08
geometry: margin=2cm
toc: true
output: pdf_document
categories: ["Networking"]
tags: ["WiFi", "Cisco"]
---

# WLAN Fundamentals

## Ad Hoc Mode

Ad Hoc Mode
:	Independent wireless devices building a wireless network without any other infrastructure

BSS
:	Basic Service Set - Area in which a given device can be reached wirelessly

IBSS
:	Independent Basic Service Set - Setup without any other infrastructure, ad hoc

## Infrastructure Mode

Infrastructure Mode
:	Wireless network built using access points 

AP
:	Access point

BSA
:	Basic Service Area (cell) - Area that an access point effectively covers

SSID
:	Service Set Identifier - 

DS
:	Distribution System - Infrastructure linking wireless system to all other network resources

WLC
:	Wireless LAN Controller - 

ESS
: Extended Service Set - When you have two or more APs on different channels, advertising the same SSID, working in conjunction via a WLC

Roaming
:	Seamless movement between acces points

* Wireless is always half-duplex

# Standards and Regulations

* Layer 1 and Layer 2 protocols
	* IEEE 802.11 working group for Wireless LANs
	* Study the apps and uses of WLANs, publish protocols, modulation, frequency, framing, physical
* Interoperability testing
	* Wi-Fi Alliance

## Regulatory Organizations

* FCC - Federal Communications Commission
	* How much power or which frequencies
* ETSI - European Telecommunications Standards Institute
* TELEC - Telecom Engineering Center (Japan)
* BRAI - Broadcasting Regulatory Authority of India

# RF Fundamentals

Frequency
:	Speed at which the wave repeats, measured in cycles per seconf (Hertz)

Wavelength
:	How wide is one cycle

Amplitude
:	Energy of waveform

RSSI
:	Received Signal Strength Indicator

* For RSSI, closer to zero is better

## Interference

Free Path Loss
:	Normal atmospheric attentuation

Scatter
:	

* Physical obstacles absorb RF
* Reflection
* Multipath results in signal cancellation
	* Upfade, downfade, or cancellation, depending on  phase
* Fade
* Long-range atmosphere refraction

* RSSI minus noise = Signal to Noise Ratio (SNR)

# SNR

* Signal - Noise floor
* E.g., -70 - (-95) = 25
* Shooting for SNR of 20 or better

# dBs and Watts
* Decibel is a comparison

* 0dBm = same
* 10dBm = 10x
* -10dBm = 0.1x
* 3dBm = 2x
* -3dBm = 0.5x

# Antennas

* H-Plane - Top down view, horizontal plane
* E-Plane - Side view

EIRP
:	Effective Isotropic Radiated Power

* FCC is 36dBm

* Antenna gain (dBi) - Focusing energy over a certain area
* Dipole reference (dBd), add 2.15dBi gain
* Unidirectional
* Omnidirectional

# Frame Types

* CSMA/CA - Collision avoidance
* WiFi is half-duplex

## 3 Categories

* Management Frames
	* Beacons, Probes, Association, Authentication
* Control Frames
	* RTS - Request to Send
	* CTS - Clear to Send
	* ACK - Confirmation of frame delivery
* Data Frames

## Slot times

* 802.11b - 20us
* 802.11a,g - 9us

NAV
:	Network Allocation Vector - Duration during which no one else can send.

SIFS
:	Short Inter Frame Space. Between 10-16us

DCF
:	

DIFS
:	Delay Inter Frame Space

# Frame Flow

# WLAN Disruptors

# WLC and AP Concepts

* Split MAC
	* AP runs radio, sends ACKs & beacons, responds to probes
	* Controller does authentication and authorization, dis-associations, policy

CAPWAP
: Control and provisioning for wireless access points

# 802.1x and EAP Concepts

## WLAN Security

Supplicant
: Provides credentials

Authenticator
: Requests credentials

Authentication Server
: Authenticates credentials

* EAP - Extensible Authentication Protocol
	* LEAP - Lightweight EAP
	* FAST - EAP Flexible Authentication via Secure Tunneling
	* PEAP - Protected EAP
		* Authenticator has certificate only
	* EAP-TLS - Transport Layer Security
		* Both supplicant and authentication server have certs
	
* Symmetrical - Same key used to encrypt and decrypt
* Assymetrical - Key pair, one to encrypt, one to decrypt
	* Public key
	* Private key
* Digital certificate contains entity public key
* Certificate Authority

# WLAN Troubleshooting

# Rogue APs

* De-auth attacks

* Radio Resource Management (RRM)

* Clean Air
	* Air Quality Index
	
* wIPS

# Frequencies and Protocols

* ISM - Industrial, Scientific & Medical band
	* 2.4Ghz, 3 non-overlapping channels
* UNII - Unlicensed National Information Infrastructure
	* 5Ghz, 
* Non Overlapping Channels
* Channel Bonding
* 802.11 Protocols
	* 802.11a
	* 802.11b
	* 802.11g
	* 802.11n
	* 802.11ac
	
# MIMO & 802.11n & 802.11ac

* MIMO - Multiple Input Multiple Output
	* Multiple antennas
* MU-MIMO - Multiple User Multiple Input Multiple Output
* Transmit x Receive : Spatial Streams
* Beamforming
* MRC - Maximal Ratio Combining

# Design Considerations

* Applications that will be used
	* Real-time - voice, video
	* Non real-time - email
* Facility details, clients, signals, APs

## Surveys

* Predictive
* Pre-survey (passive and active)
* Post-survey
