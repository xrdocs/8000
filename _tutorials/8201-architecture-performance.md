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
  - Performance
  - Scale
---
{% include toc icon="table" title="Cisco 8201 High Level Architecture and Performance" %} 

## Introduction

Released 2 years ago in late 2019, Cisco 8201 router has been widely adopted and deployed by service provider and cloud customers for core and peering roles. Based on Cisco Silicon One Q100 NPU, the first generation of this new chip family, it provides both 400GE and 100GE port options which make it perfect for peering role in a small form factor.  

This short article will first cover Cisco 8201 high level internal architecture. Then, a quick and simple lab test will demonstrate how it can achieve 10.8Tbps at target Non-Drop Rate (NDR). This test will address recent concerns raised about its forwarding performance.

## Cisco 8201 High Level Architecture
Cisco 8201 is the first Cisco 8000 fixed system built around a single NPU. It has 24 x QSFP56-DD 400GE and 12 x QSFP28 100GE ports, all packed in a compact 1RU form factor. Datasheet is available [here](https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/datasheet-c78-742571.html "Cisco 8201 datasheet").  

At the heart of Cisco 8201 is the Cisco Silicon One Q100 ASIC. This NPU is capable of forwarding 10.8Tbps at a rate of 6.6Bpps. It’s made of 216 x 56G SerDes, which can be configured for differing port speeds of 10G/25G/50G using NRZ or PAM4 modulation.  

The Q100 NPU architecture is built around 6 slices. Each slice handles the forwarding and feature processing for a set of physical interfaces.  All of the slices are in turn interconnected by a Shared Memory Switch (SMS).  

![8201-arch.png]({{site.baseurl}}/images/8201-arch.png){: .align-center}

On a fixed system like Cisco 8201, the 6 slices are operating as ‘System on a Chip’ or ‘Router on a Chip’, meaning all slices interconnect network facing interfaces:  

<img class="centered" src="{{site.baseurl}}/images/router-on-chip.png" width="50%"/>{: .align-center}

Each slice has:
- Receive and Transmit Network Processing Engines (Rx NPE  and Tx NPE) 
- 2 x Interface Group (IFG)  

On the Q100, each IFG has 18 x 56G SerDes, providing 900Gbps of interface bandwidth. Each slice can forward one packet per clock cycle, so a system build from one Q100 NPU can thus forward six packets per clock.  

![Q100-HD.png]({{site.baseurl}}/images/Q100-HD.png){: .align-center}

Let’s recap and do the math:
- 6 slices x 2 x IFG x 900Gbps = 10.8Tbps of available interface bandwidth
- 6 slices x 1.1Bpps = up to 6.6Bpps total switching rate.  

A combination of HBM, SRAM and TCAM memories are used for packet buffering and database storage.  

Add a quad-core Intel CPU, 32GB of RAM and a 64GB SSD and you have a Cisco 8201.

The mapping of individual ports to slices/IFG can be checked with following command:  

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:8201#sh controllers npu voq-usage interface all instance 0 location 0/RP0/CPU0
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
RP/0/RP0/CPU0:8201#</code>
</pre>
</div>

## Lab topology
A Cisco 8201 has been staged in the lab with the latest IOS-XR version available when writing this article (7.3.2). This router is connected to 24x400G and 12x100G dedicated Spirent ports. Spirent is used to send traffic through the router at a specific packet size (200 bytes). It’s also used to measure total packet throughput (Gbps), frame rate (pps), packet loss and system latency.  

Spirent configuration is attached for reference:  
![spirent-config.png]({{site.baseurl}}/images/spirent-config.png){: .align-center}

## Results
As shown in the [video](https://www.youtube.com/watch?v=TUKOQTx2GC8), the Cisco 8201 is capable of forwarding the full 10.8Tbps of traffic composed of 200bytes IPv4 packets, at the sustained rate of 6Bpps.

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/TUKOQTx2GC8" frameborder="0" allowfullscreen></iframe>{: .align-center}
  
This is achieved without any single packet loss, as shown in below screenshots:  

![8201-pps.png]({{site.baseurl}}/images/8201-pps.png){: .align-center}

  
![8201-bps.png]({{site.baseurl}}/images/8201-bps.png){: .align-center}

![8201-no-packet-loss.png]({{site.baseurl}}/images/8201-no-packet-loss.png){: .align-center}




<a href="{{site.baseurl}}/images/NDR-no-drop.png">
  <img class="centered" src="{{site.baseurl}}/images/NDR-no-drop.png"/>{: .align-center}
</a>  

Average system latency is 3.3µs. Same NDR is observed for IPv6 and MPLS traffic.

## Conclusion
This article covered Cisco 8201 high level architecture. A simple lab test demonstrated raw Cisco 8201 forwarding capacities. Current NDR for this platform is 200 bytes for IPv4, 200 bytes for MPLS and 200 bytes for IPv6. This test confirmed Cisco Q100 performance (10.8Tbps, 6Bpps).  
Seeing is believing: [Cisco 8000 CPOC lab](https://dcloud-docs.cisco.com/c/r/dcloud-docs/sites/en_us/explore/mig/8000-lab.html) is available on [dcloud](https://dcloud-docs.cisco.com/c/r/dcloud-docs/sites/en_us/explore/mig.html). Book the lab with your account team, repeat the tests yourself, too.  
A next article will cover Cisco 8201 FIB scale and performance in real life conditions.
