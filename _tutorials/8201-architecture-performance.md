---
published: true
date: '2021-12-09 15:18 +0100'
title: Cisco 8201 High Level Architecture and Performance
author: Frederic Cuiller
excerpt: >-
  This short article will first cover Cisco 8201 high level internal
  architecture. Then, a quick and simple lab test will demonstrate how it can
  achieve 10.8Tbps at target Non-Drop Rate (NDR). This test will address recent
  concerns raised about its forwarding performance.
position: hidden
tags:
  - iosxr
  - Cisco 8000
---
{% include toc icon="table" title="Cisco 8201 High Level Architecture and Performance" %} 

## Introduction

Released 2 years ago in late 2019, Cisco 8201 router has been widely adopted and deployed by service provider and cloud customers for core and peering roles. Based on Cisco Silicon One Q100, first generation of this new chip family, it provides both 400GE and 100GE port options.  

This short article will first cover Cisco 8201 high level internal architecture. Then, a quick and simple lab test will demonstrate how it can achieve 10.8Tbps at target Non-Drop Rate (NDR). This test will address recent concerns raised about its forwarding performance.

## Cisco 8201 High Level Architecture
Cisco 8201 is the first Cisco 8000 fixed system built around a single NPU instance. It has 24 x QSFP56-DD 400 GbE and 12 x QSFP28 100 GbE ports, all packed in a compact 1RU form factor. Datasheet is available on this [link](https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/datasheet-c78-742571.html "Cisco 8201 datasheet").  

At the heart of Cisco 8201, there is Cisco Silicon One Q100 ASIC. This NPU is capable of forwarding 10.8Tbps at a rate of 6.6Bpps. It’s made of 216 x 56G SerDes, each can be configured independently to operate in 10G/25G/50G using NRZ or PAM4 modulation.  

The NPU is built around 6 slices. All slices are interconnected by a Shared Memory Switch (SMS). 

![Q100.png]({{site.baseurl}}/images/Q100.png)

On a fixed system like Cisco 8201, the 6 slices are operating as ‘System on a Chip’ or ‘Router on a Chip’, meaning all slices interconnect network facing interfaces:  

<img class="centered" src="{{site.baseurl}}/images/router-on-chip.png" width="50%"/>

Each slice has:
- Receive and Transmit Packet Processor (RxPP and TxPP)
- 2 x Interface Group (IFG)  

In addition, each IFG has 18 x 56G SerDes, providing 900Gbps per IFG.

Last, each slice is clocked at 1.1GHz, meaning it can handle up to one packet per clock.

Let’s recap and do the math:
- 6 slices x 2 x IFG x 900Gbps = up to 10.8Tbps total bandwidth IO
- 6 slices x 1.1Bpps = up to 6.6Bpps total switching rate.  

HBM, SRAM and TCAM memories are used for packet buffering and databases storage. A dedicated article will cover those different memories, how they are used and what sizes they are.

Add a quad-core Intel CPU, 32GB of RAM and a 64GB SSD and you have a Cisco 8201: 
![8201-architecture.png]({{site.baseurl}}/images/8201-architecture.png)

The mapping of individual ports to slices/IFG can be checked with following command:  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201#sh controllers npu voq-usage interface all instance 0 location 0/RP0/CPU0
Tue Dec  7 09:50:16.205 PST

-------------------------------------------------------------------
Node ID: 0/RP0/CPU0
Intf         Intf     <mark>NPU Slice IFG</mark>  Sys   VOQ   Flow   VOQ    Port
name         handle    <mark>#    #    #</mark>   Port  base  base   port   speed
             (hex)                                      type
----------------------------------------------------------------------
FH0/0/0/0    f000138   0   0    0     4    448      0 local   400G
FH0/0/0/23   f000140   0   1    0    96    464      0 local   400G
FH0/0/0/22   f000148   0   1    0    92    472      0 local   400G
FH0/0/0/21   f000150   0   1    1    88    480      0 local   400G
FH0/0/0/20   f000158   0   1    1    84    488      0 local   400G
FH0/0/0/19   f000160   0   2    0    80    496      0 local   400G
FH0/0/0/18   f000168   0   2    0    76    504      0 local   400G
FH0/0/0/17   f000170   0   2    1    72    512      0 local   400G
FH0/0/0/16   f000178   0   2    1    68    520      0 local   400G
FH0/0/0/15   f000180   0   3    0    64    528      0 local   400G
FH0/0/0/14   f000188   0   3    0    60    536      0 local   400G
FH0/0/0/13   f000190   0   3    1    56    544      0 local   400G
FH0/0/0/12   f000198   0   3    1    52    552      0 local   400G
FH0/0/0/1    f0001a0   0   0    0     8    456      0 local   400G
FH0/0/0/2    f0001a8   0   0    1    12    560      0 local   400G
FH0/0/0/3    f0001b0   0   0    1    16    568      0 local   400G
FH0/0/0/4    f0001b8   0   5    0    20    576      0 local   400G
FH0/0/0/5    f0001c0   0   5    0    24    584      0 local   400G
FH0/0/0/6    f0001c8   0   5    1    28    592      0 local   400G
FH0/0/0/7    f0001d0   0   5    1    32    600      0 local   400G
FH0/0/0/8    f0001d8   0   4    1    36    608      0 local   400G
FH0/0/0/9    f0001e0   0   4    1    40    616      0 local   400G
FH0/0/0/10   f0001e8   0   4    0    44    624      0 local   400G
FH0/0/0/11   f0001f0   0   4    0    48    632      0 local   400G
Hu0/0/0/24   f0001f8   0   3    0   100    640      0 local   100G
Hu0/0/0/35   f000200   0   4    1   144    656      0 local   100G
Hu0/0/0/34   f000208   0   4    0   140    672      0 local   100G
Hu0/0/0/33   f000210   0   5    1   136    688      0 local   100G
Hu0/0/0/32   f000218   0   5    0   132    704      0 local   100G
Hu0/0/0/31   f000220   0   0    1   128    720      0 local   100G
Hu0/0/0/30   f000228   0   0    0   124    736      0 local   100G
Hu0/0/0/29   f000230   0   1    1   120    752      0 local   100G
Hu0/0/0/28   f000238   0   1    0   116    768      0 local   100G
Hu0/0/0/27   f000240   0   2    1   112    784      0 local   100G
Hu0/0/0/26   f000248   0   2    0   108    800      0 local   100G
Hu0/0/0/25   f000250   0   3    1   104    816      0 local   100G
RP/0/RP0/CPU0:8201#
</code>
</pre>
</div>

## Lab topology
A Cisco 8201 has been staged in the lab with the latest IOS-XR version available when writing this article (7.3.2). This router is connected to 24x400G and 12x100G dedicated Spirent ports. Spirent is used to send traffic through the router at a specific packet size (200 bytes). It’s also used to measure total packet throughput (Gbps), frame rate (pps), packet loss and system latency.  

Spirent configuration is attached for reference:  
![spirent-config.png]({{site.baseurl}}/images/spirent-config.png)

## Results
As shown in below [video](https://www.youtube.com/watch?v=TUKOQTx2GC8), Cisco 8201 is capable to forward 10.8Tbps of traffic composed of 200bytes IPv4 packets. It’s able to sustain a rate of 6Bpps full duplex:  

[![Cisco 8201 NDR & raw performance](https://img.youtube.com/vi/TUKOQTx2GC8/0.jpg)](https://www.youtube.com/watch?v=TUKOQTx2GC8){: .align-center}

This is achieved without any single packet loss, as shown in below screenshot:  
<a href="{{site.baseurl}}/images/NDR-no-drop.png">
  <img class="centered" src="{{site.baseurl}}/images/NDR-no-drop.png"/>
</a>  

Average system latency is 3.3us. Same NDR is observed for IPv6 and MPLS traffic.

## Conclusion
This article covered Cisco 8201 high level architecture. A simple lab test demonstrated raw Cisco 8201 forwarding capacities. Current NDR for this platform is 200 bytes for IPv4, 200 bytes for MPLS and 200 bytes for IPv6. This test also confirmed Cisco Q100 performance (10.8Tbps, 6Bpps).  
Seeing is believing: Cisco 8000 CPOC lab is available on dcloud. Book the lab with your account team, repeat the tests yourself, too.  
Next articles will cover Cisco 8201 FIB scale and performance in real life conditions.
