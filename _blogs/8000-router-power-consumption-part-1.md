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

## Metrics for the Cisco 8000 Series 

At the beginning of this paper, the metric of watts per 100G without optics was proposed. Let’s conclude by looking at those values for some specific configurations. The data below is tested at 22C with 768B packets, 15% bandwidth, and without optics. 

| System              | Bandwidth  | Typical Power (W) | Watts / 100G |
| ------------------- | ---------- | ----------------- | ------------ |
| 8201-32FH           | 12.8 Tbps  | 265               | 2.1          |
| 8804 (4x 14.4T LC)  | 43.2 Tbps  | 4200              | 7.2          |
| 8808 (8x 14.4T LC)  | 86.4 Tbps  | 8100              | 7.0          |
| 8812 (12x 14.4 LC)  | 172.8 Tbps | 13900             | 8.1          |
| 8818 (18x 14.4T LC) | 259.2 Tbps | 18900             | 7.6          |
