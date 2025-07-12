---
title: "TCP Overview"
date: '2024-10-24'
author: Craig Bruenderman
geometry: margin=2cm
type: list
output: pdf_document
image: /images/tcp-ip-model.webp
---

# TCP

## TCP Handshake

* Critcal to capture the handshake since only it has window scale factor
* Also indicates the initial round trip time
  * If client side capture, between SYN and SYN ACK
  * If server side capture, between SYN ACK and ACK
  * If somewhere in between, you can add delta times together

### SYN

* SYN length is 0
* Window size set
* Window scaling set, if applicable (should be)
* Sender MSS will be set

### SYN ACK

* SYN ACK will ACK 1 "ghost byte"
* SYN and ACK bits set
* Receiver MSS will be set
  * Lower of the two will be used

### ACK

* SYN not set, ACK only

## Sequence Numbers

* Used to track bytes in each direction of a connection
* Next sequence number will be sequence number, plus length
* Previous Segment Not Captured indicates gap in TCP sequence numbers
  * Either dropped, or will come in out of order

## Acknowledgment Numbers

* Normally, server should ACK the SEQ number
* ACK will be repeated in subsequent data just indicating nothing new was acknowledged
* Wireshark will indicate checkmark on far left of packet being ACKed

# TCP Windows

* The sender of a packet will advertise a window
  * This is the senders receive window
