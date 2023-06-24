---
published: true
date: '2023-06-16 00:50 +0200'
title: Cisco 8608 Architecture
position: hidden
excerpt: >-
  This post will describe Cisco 8000 centralized system architecture: Cisco
  8608, based on Silicon One Q200 ASIC.
author: Chang Soo Lee
tags:
  - Cisco 8000
  - Centralized System
  - Silicon One
---
{% include toc icon="table" title="Cisco 8608 Architecture" %}

## Introduction
The Cisco 8608 series is a part of Cisco’s Centralized Architecture based chassis design. Centralized systems have redundant Route Processors (RP w/CPU), redundant Switch cards (SCs w/Cisco Silicon One Q200 NPU) and in-service replaceable modular port adapters (MPA). Unlike distributed architecture, the forwarding decisions on Centralized platforms are centrally performed at the RP/SC and runs Cisco IOS XR software.  

![Figure1.png]({{site.baseurl}}/images/Figure1.png){: .full}  
Figure 1. Front view of the Cisco 8608  
{: .text-center}  

![figure2.png]({{site.baseurl}}/images/figure2.png){: .align-center}  
Figure 2. Front view of the Cisco 8608 with Door kit  
{: .text-center}  

The Cisco 8608 chassis is 7RU in height and less than 600mm in sheet metal depth with 8 slot for three types of MPAs(Modular Port Adapter).  

Supporting the 8608 Series capabilities demanded a wide range of new approaches to platform design. By leveraging over decades of high-performance system design, Cisco has delivered unprecedented capacity without compromising forwarding performance or requiring oversubscription. This required new power supplies, a redesigned cooling architecture, and future-proof connectors.  
The Cisco 8608 router required extensive innovation and consolidation of capabilities of multiple ASICs into a single ASIC to greatly increase the power efficiency of the solution. As with all modern networking devices the power density of the ASIC creates a challenging thermal situation. These challenges are addressed via advanced system design, including futuristic fans and heat sinks in Cisco 8608.  
The 8608 chassis utilizes a state-of-the-art orthogonal design with advanced cooling via front to back airflow, high power capacity and new power supplies. The chassis and all data path components for the 8608 Series benefit from a clean sheet design that allows the systems to take full advantage of the latest technologies and Cisco’s design expertise. This design connects all forwarding path components directly without a backplane or midplane. In the 8608 Series, the eight Modular Port Adaptors (MPAs) along with the two Route Processors are vertically oriented. Every major component of the 8608 Series was developed with a clean sheet approach – representing unprecedented investment and commitment to a long lifecycle for the 8608 Series. 
Investment Protection – The design of the system keeps in mind the ability to upgrade all the major components of the Chassis (Switch Card, Route Processor & Modular Port Adaptors). Cisco is investing in the future of the platform for the customers to be able to upgrade to next generation.  

Cisco 8608 is comprised of the following components:  
•	Chassis: 7RU modular chassis which hosts commons (fans, power supplies), Route Processors (RP), Switch Cards (SC) and Modular Port Adapters (MPAs).  
•	Route Processor (RP): Cisco 8608 RP hosts the CPU and the I/O ports on it. RPs fit vertical in the chassis and are inserted from the front panel. There will be up to two redundant RPs in the system and the RPs will operate in Active-Standby mode for a redundant system.
•	Switch Card (SC): Switch cards sit behind the MPAs and have connections to all MPAs. Switch cards host the NPUs on them. There are up to two switch cards in the system and the SCs work in Active-Standby mode thus delivering data plane redundancy. 
•	Power Supplies: Cisco 8608 has 4 power supplies that can provide N+N redundant power to the system. The available options are:  
o	Pluggable 3.2KW AC 
o	Pluggable 3.2KW DC 
•	Fans: There are 8 fans in the system with each fan individually removable / replaceable. These provide N+1 fan redundancy to the system. The entire fan tray can be removed in case of SC insertion or removal.  

## Hardware Description  
### Chassis  


![]({{site.baseurl}}/images/figure3.png)![figure3.png]({{site.baseurl}}/images/figure3.png){: .align-center}  

Figure 3. Front view of an empty Cisco 8608 chassis  
{: .text-center}  

Cisco 8608 chassis utilizes a state-of-the-art orthogonal design with advanced cooling via front to back airflow, high power capacity and new power supplies. The chassis and all data path components.  

As shown in Figure 3, user needs to insert the followings:  
•	Modular Port Adapter(MPA) sits vertically in the front of the chassis
•	Route Processor(RP) sits vertically in the front of the chassis
•	Switch Card(SC) sits horizontally behind the Fan Trays. Cisco 8608 chassis has two Fan Trays with four Fan Modules on each Fan Tray
•	Midplane provides the control path connectivity and power distribution for the chassis


![]({{site.baseurl}}/images/figure4.png)![figure4.png]({{site.baseurl}}/images/figure4.png){: .align-center}  

  Figure 4. Back view of Cisco 8608 chassis with 8 FMs and 4 PMs  
  {: .text-center}  
  In Figure 4, there are 2 Fan Trays with eight Fan Modules and four Power Modules  
  
  

## RP

## Switching Card

## MPA

## Redundancy

## Conclusion
