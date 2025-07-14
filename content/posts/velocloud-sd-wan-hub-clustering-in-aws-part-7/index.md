---
title: "VeloCloud SD-WAN Hub Clustering in AWS (Part 7)"
draft: true
date: "2025-07-06"
series: ["VeloCloud SD-WAN Hub Clustering in AWS"]
series_order: 7
categories: ["Networking"]
tags: ["AWS", "SD-WAN", "Transit Gateway", "Velocloud"]
---

## Part 7: Troubleshooting

As we can see here, the ENI mapped to GE1 which initially had a public IP no longer has one. This is due to EC2 EIP behavior which released the public IP when I stopped the Instance, and did not re-acquire one on that particular ENI since another public IP is already assigned to another interface (the ENI for GE2). I believe that would have happened even if I hadn't assigned an EIP to GE2 because of the rule related to an instance having multiple ENIs.

Hmm, Connection refused. I believe this is related to iptables rules on the Edge itself. Initially, it seems to have no rules for its GE1 interface, as I was initially able to SSH to that interface right after launch.