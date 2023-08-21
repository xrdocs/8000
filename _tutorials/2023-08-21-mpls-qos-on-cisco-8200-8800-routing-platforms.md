---
published: false
date: '2023-08-21 12:47 +0530'
title: MPLS QoS on Cisco 8200/8800 routing Platforms
author: Ram Mohan A. M.
excerpt: This writeup explains the MPLS-QOS applications on Cisco 8000 router
tags:
  - iosxr
---

## Introduction
Diffserv Tunnelling Modes introduces a new Per-Hop-Behaviour (PHB), which allows differentiated QoS in a provider’s network. The tunnelling mode is defined at the edge of the network, normally in the PE label switch routers (LSRs) (both ingress and egress). You may need to make changes in the P routers; you must also consider what occurs when the topmost label is removed from a packet due to Penultimate-Hop-Popping (PHP). It may be necessary to copy the MPLS EXP value from the top label that is being popped to the newly exposed label; this does not always apply to all tunnelling modes.

## PHP Vs Diffserv
In some cases (for example, a plain non-VPN MPLS network), the PHP action on the final P router can expose a plain IP packet when a packet with only one label is received. When this IP packet is received by the egress LSR (PE), it is not possible to classify the packet based on the MPLS EXP bits because there is no label now. In these situations, you must configure the egress PE router to advertise an explicit-null label. When the PHP action is performed on the P router, a label with a value of zero is sent, and with this special label you can mark the EXP bits as normally labelled packets, allowing the correct classification on the egress PE router.
The MPLS network support of Diffserv specification defines these tunnelling modes:
  - Uniform
  - Pipe
  - Short-Pipe


## IP Precedence/DSCP to MPLS-EXP mapping

![Screenshot 2023-08-21 at 4.11.37 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-21 at 4.11.37 PM.png)







