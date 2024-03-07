---
published: true
date: '2024-03-07 15:01 +0100'
title: Cisco Live Melbourne 2023 – Cisco 8000 800G Performance Demo
author: Frederic Cuiller
tags:
  - Cisco 8000
  - 800G
  - Performance
  - Demo
position: hidden
excerpt: >-
  This video first introduces the reason why industry needs 800G. It then covers
  the key ingredients needed to build dense 800G systems: SerDes, ASICs, optics
  and standards. It also explains the differences between 800G and 800GbE and
  800GbE development and deployment timeline. Last, this video shows a live demo
  of a 36x800G line card delivering 28.8Tbps full duplex traffic, on all ports,
  linerate, inside a 8808 distributed system.
---
# Introduction

This video first introduces the reason why industry needs 800G. It then covers the key ingredients needed to build dense 800G systems: SerDes, ASICs, optics and standards. It also explains the differences between 800G and 800GbE, with focus on IEEE 800GbE development and deployment timeline. Last, this video shows a live demo of a 36x800G line card delivering 28.8Tbps full duplex traffic, on all ports, linerate, inside a 8808 distributed system.

# Demo Details

This silent/audio-less video was recorded for Cisco Live Melbourne 2023 and was the support of a World of Solutions demo.

The test is done on a Cisco 8808 chassis. It contains latest RP2 route processor, 8808-FC1 fabric cards and a 88-LC1-36EH line card.  

To demonstrate the line card performance, a common technique is to use a 'snake topology' where ports are wired together. Cabling is done in a way to perform inter-NPU forwarding and thus send traffic over the fabric.  

Multiple QDD-2X400G-FR4 modules are used and create individual FourHundredGigabitEthernet interfaces in IOS XR, for a total of 72 x 400GbE interfaces. Two of those ports are connected to a traffic generator.

Traffic generator is configured to send a simple stream of IPv4, IPv6, or MPLS traffic. Purpose of the test is to demonstrate raw performance. Streams are configured to send 100% line rate traffic with a fixed packet size to demonstrate the line card Non Drop Rate (NDR). In this test, the NDR is achieved with a packet size of 296B. It is the same for IPv4, IPv6 and MPLS. 

# Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/YJGFu-Xkfxs?si=TcEdAZR9X-G_L1Qw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# Conclusion

800G is real and Cisco 8000 is ready. Beware of confusion with 800GbE: remember a whole ecosystem is needed to have mass adoption. For the next few years, 800G deployments will be mostly based on 2 x 400GbE. History repeats, like 400GbE & 100GbE few years back.

# Acknowledgement

I'd like to thank Saurabh Maheshwari for the support provided during this demo.
