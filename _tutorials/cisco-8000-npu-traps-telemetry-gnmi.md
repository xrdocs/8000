---
published: true
date: '2022-04-20 21:17 +0200'
title: Cisco 8000 NPU traps collection with telemetry
position: hidden
tags:
  - iosxr
  - Cisco 8000
  - gNMI
  - NPU
  - traps
author: Frederic Cuiller
excerpt: >-
  On Cisco 8000, Network Processing Unit (NPU) traps are raised in response to
  the type of packets received by the router. This post will describe how to
  retrieve Cisco 8000 NPU traps counters through telemetry using gNMI interface.
---
{% include toc icon="table" title="Cisco 8000 NPU traps collection with telemetry" %} 

## Introduction
On Cisco 8000, Network Processing Unit (NPU) traps are raised in response to the type of packets received by the router. Monitoring those counters can be interesting to detect packet loss or to measure deviation from production baseline for example.  

This post will describe how to retrieve Cisco 8000 NPU traps counters through telemetry using gNMI interface.

## Cisco 8000 NPU traps
There are 3 NPU traps categories:

1. Exception packets. For example, glean adjacency traffic or packets with IPv4 options.
2. Error packets. For example, IPv4 packet with bad checksum or IPv6 packets with a hop count of zero.
3. non-LPTS control packets. For example, packets that do not get processed through LPTS (like LACP, LLDP and other L2 control packets).

Each of the NPU traps are policed at a rate that is pre-programmed by the operating system. Packets are policed per NPU within the router and there can be several NPU per system. Some NPU trap packets allowed by NPU policers are sent to the CPU (Linecard or RP CPU for distributed systems, RP CPU for fixed and centralized systems) if they need additional processing. Others that exceed the NPU policer rate are simply dropped by the NPU.  

NPU trap statistics can be retrieved with following CLI:
```
show controllers npu stats traps-all instance NPU-Number|all location RP|LC
```

It’s important to understand the system architecture to know which NPU instance should be checked, especially when dealing with linecards in Cisco 8800 distributed systems. [Following](https://xrdocs.io/8000/tutorials/cisco-8000-port-assignment/ "Port Assignments on Cisco 8100/8200 and Cisco 8800 Platforms") article written by CS Lee covers port assignment and will help identifying NPU instance for a given interface.
{: .notice--info}

