---
published: true
date: '2022-03-07 15:40 +0100'
title: 'Router Power Consumption (part 1) '
author: Lane Wigley
excerpt: >-
  Power claims for networking equipment are often difficult to understand. It’s
  not always clear whether chip, card, or system power is being stated and what
  conditions were used for the evaluation. Let’s take a step back and consider a
  better way of keeping score. 
tags:
  - iosxr
  - Cisco 8000
  - Power
position: hidden
---
{% include toc icon="table" title="Router Power Consumption (part 1)"%}

Power claims for networking equipment are often difficult to understand. It’s not always clear whether chip, card, or system power is being stated and what conditions were used for the evaluation. Let’s take a step back and consider a better way of keeping score. In my opinion – hopefully soon yours too – these are the day-to-day system power consumption and the provisioned power (number and capacity of PSUs to support redundancy and worst-case conditions). This post will focus on exploring the components of system level power consumption. Part 2 of this series will investigate power provisioning. Part 3 will discuss setting up a test environment to standardize this evaluation and results with the Cisco 8000 Series.  

Let’s start by defining a unit for the metric. I propose “watts per 100G without optics” based on system power consumption. It represents a common unit of bandwidth and (for high end routers) currently yields numbers in the 1-50W range that most of us are comfortable with. Optics should be subtracted because they are highly variable based on module type (e.g., 400G ZR+ is twice the power of FR4). As optics may impact cooling requirements, a more specific definition of “Watts per 100G at typical temperature with 2km optics minus optics module power” is a good benchmark. Variations on this, such as ZR+ optics or fewer cards or ports, may also be defined for operator-specific comparisons.

## System Power
System power consumption is the key metric to focus on. Chip power is an interesting way to compare devices, but it’s not relevant when it comes to paying the power bill or assessing environmental impact. Let’s look at what makes up system power by reviewing the following key components:

- Electrical data path
- Optics
- Cooling
- Mechanical design
- Control plane
- Losses in voltage conversion and signaling

## Electrical Data Path 

The key components of the electrical data path power comprise PHYs, retimers, and forwarding ASICs. PHYs are small devices used for functions such as MACsec or as gearboxes which combine SerDes (Serializer/Deserializers are used for fast I/O through PCBs) or split them into different speeds. A common gearbox operation is to convert 2x 56G SerDes from the ASIC to 4x 28G SerDes connected to a QSFP28 100 GbE port. Retimers are used to extend electrical signals. Retimers are commonly used for long traces on larger PCBs (e.g., 8818 fabric card) and to connect optics to ASICs in the “back row” of a line card.  
![Linecard PHY]({{site.baseurl}}/images/linecard-phys.png)
Fig 1. PHYs on a line card.
{: .text-center}  
Power-wise, ASICs are the stars of the show, so it’s understandable that they get most of the attention. They are the most complex components and are the key driver of system power for large, modular systems like the 8800. ASIC power can be broken into three main categories: SerDes, idle core power, and dynamic core power. In high-bandwidth chips like the Cisco Silicon One Q200, these are roughly equal. I’ll be focusing on this class of chips in this paper. It includes devices such as Broadcom’s Jericho 2, Jericho 2c+, and Tomahawk line as well as Juniper’s Triton and the recently preannounced Express 5. Edge-class chips such as Cisco’s Lightspeed Plus and Juniper’s Trio have similar characteristics but consume more power in complex per-packet operations (e.g., more counters) and less in SerDes due to lower bandwidth.  

Each of these categories has different characteristics of power consumption. SerDes require constant power independent of traffic load, but they can be powered down to save power. On the 8000 Series, this occurs when ports are shut down or when a fabric or line card slot is empty. In the latter case, the corresponding SerDes on the connecting card are shut down (e.g., some Fabric Card (FC) SerDes are disabled when Line Card (LC) slots are empty).  

ASIC idle power is exactly what it sounds like. It’s the power required to keep the chip’s internal blocks up and running with no traffic load. Dynamic power occurs when packets are forwarded. Some dynamic power is consumed for per-packet operations (pps load) and some is needed for moving the bandwidth along the forwarding path (bps load).  

Core (non-SerDes) ASIC power will vary greatly among devices as networking ASICs serve many roles with a variety of requirements. Core ASIC power may vary based on several factors:  
-Function – DC vs. SP Edge vs. SP Core perform different levels of per-packet operations 
-Network vs. fabric capacity – ASIC interfaces that only can connect to a switch fabric are much simpler and lower power 
-Process node (7nm is the current generation, 16nm and 28nm were the previous TSMC nodes) 
-Design – Newer microarchitectures tend to have better efficiency than chips carrying legacy constraints. The skill of the design engineers is also significant and shows up in the power bottom line. 
-Device temperature – A hot ASIC has marginally higher power. 
-Load – The number of transistors flipping at a given time (mainly a function of pps and bandwidth, but also of the design’s ability to gate unused logic)  


## Metrics for the Cisco 8000 Series 

At the beginning of this paper, the metric of watts per 100G without optics was proposed. Let’s conclude by looking at those values for some specific configurations. The data below is tested at 22C with 768B packets, 15% bandwidth, and without optics. 

| System              | Bandwidth  | Typical Power (W) | Watts / 100G |
| ------------------- | ---------- | ----------------- | ------------ |
| 8201-32FH           | 12.8 Tbps  | 265               | 2.1          |
| 8804 (4x 14.4T LC)  | 43.2 Tbps  | 4200              | 7.2          |
| 8808 (8x 14.4T LC)  | 86.4 Tbps  | 8100              | 7.0          |
| 8812 (12x 14.4 LC)  | 172.8 Tbps | 13900             | 8.1          |
| 8818 (18x 14.4T LC) | 259.2 Tbps | 18900             | 7.6          |  


