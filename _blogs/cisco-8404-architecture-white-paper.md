---
published: true
date: '2025-04-22 12:28 +0530'
title: Cisco 8404 Architecture White Paper
author: Paban Sarma
excerpt: >-
  This post will describe Cisco 8404 compact centralised platform architecture
  and capability deep dive.
tags:
  - iosxr
  - cisco
  - Silicone One
position: hidden
---
{% include toc icon="table" title="Cisco 8608 Architecture White Paper" %}

## Cisco 8000 Overview   
The Cisco® 8000 Series combines the revolutionary Cisco Silicon One™, IOS XR® software, and clean sheet chassis design to deliver a breakthrough in high-performance routers. The 8000 Series comprises a full range of feature-rich and highly scalable and reliable routers. Cisco 8404 is a part of a new complementary portfolio built on Centralized Architecture based chassis design in a very compact form factor.  

The Cisco 8404 is a unique platform that combines flexibility and reliability while offering investment protection. Customers can achieve unmatched reliability with redundant control and data plane via redundant route switch processors. Unlike distributed architecture, the forwarding decisions on centralized platforms are centrally performed. Cisco 8404 can also be operated in a non-redundant mode as per the appropriate customer required use case. Cisco 8404 first generation route switch processor cards are powered by Cisco SiOne K100 NPUs. A wide variety of modular port adapters allow for high interface diversity.  

This detailed white paper is specifically meant for the Cisco 8404 router. In addition to the centralized router architecture in the Cisco 8404 (as well as Cisco 8608), the Cisco 8000 Series includes two other distinct router architectures that utilize the Cisco Silicon One NPU:  
  - **Distributed**: The Cisco 8800 Series provides the highest bandwidth via distributed chassis with a redundant control plane and switch fabric. The 8800 Series includes the Cisco 8804, 8808, 8812, and 8818. These chassis deliver up to 28.8 Tbps per line card via 100, 400, and 800G ports.
Learn more about [Cisco 8000 distributed] (https://www.cisco.com/site/us/en/products/networking/sdwan-routers/8000-series/index.html#accordion-68bac828a9-item-ffa3339b33) portfolio.

  - **Fixed**: The Cisco 8010, 8100 and 8200, 8700 utilize Cisco’s standalone architecture to deliver full routing functionality with a single NPU per router. Both support the full routing feature set.
Learn More about [Cisco 8000 fixed](https://www.cisco.com/site/us/en/products/networking/sdwan-routers/8000-series/index.html#accordion-68bac828a9-item-a3433fccdd) portfolio. 

## Cisco 8408 Overview   
Large, distributed chassis have traditionally been designed to cater to the market requirement for total system bandwidth, port diversity, and redundancy. The requirements far exceeded what could be accomplished with a single NPU. The fabrics that weave together multi-NPU systems had to be provisioned to the highest bandwidth use case and deployed 100% upfront, consuming power and capital regardless of the number of installed line cards. That wisdom held true for multiple generations of core, edge, and aggregation distributed systems.  

Now with the advent of dense Cisco Silicon One NPU, we can address these roles with a new breed of centralized architectures that combine the best aspects of distributed and fixed systems.  Cisco 8404 has combined the redundancy and I/O diversity of distributed systems with the economics and simple elegance of fixed platforms in a very compact 4RU, <300 mm depth form factor. It also offers extended temperature range of operation making it suitable for ruggadized environment deploymnet in the Service Provider netwroks. 
  
     
![8404-front.png]({{site.baseurl}}/images/8404-front.png)
Figure 1. Front view of the Cisco 8408  
{: .text-center} 


The Cisco 8404 router is 4 RU in height and less than 300mm in depth with 4 slots for two types of MPAs (Modular Port Adapters) at the FCS. There is also an integrated port adpter built into the Route Switch Processor (RSP) card which acts like the modular port adapters in terms of forwarding and redundancy behaviour.   


| Items |         Details       |
|:---:| :------------:|
| Base Chassis | The base Chassis is equiped with slots for all FRU and the backplane to interconnect them |
| Route Switch Processor (RSP)| The RSP card is heart of the ssytem that does all the control plane and data plane functionality as well as houses an inbuilt port adapter|
| Power Entry Modules (PEM) | The DC only PEMs inputs and supplies power to the RSPs and the entire system|
| Fan Tray | The Fan Tray houses redundnat FANs responsible for system colling|
| Modular Port Adpater (MPA)|Apart from the ports at the RSP there are four slots for Modular port adapters enabling the system to offer flexible speed/feed combination|
   
Table 1. Cisco 8404 key components      
{: .text-center}  

### Cisco 8404 value proposition  

- **Redundancy**: Cisco 8404 introduces a new forwarding paradigm via a redundant control & data plane (route switch processor [RSP]  redundancy) in a very compact form factor. A redundant 8404 requires two RSPs, while a nonredundant 8404 is comprised of a single RSP. This redundant architecure allows control and data plane to converge with minimal loss (<50ms) during failover scenario.  

- **Investment Protection**: The design of the system keeps in mind the ability to upgrade all the major components of the Chassis (Switch Card, Route Processor and Modular Port Adaptors). Cisco is investing in the future of the platform for customers to be able to upgrade to the next generation.  

- **Flexibility**: Flexibility via MPA modularity and interface diversity. Native port speed support from 1 GbE upto 400 GbE. Wide variety of optics support for multiple customer use cases. Based on speed and feed requirements the chassis can be configured with MPA combinations.

- **Innovation**: The 8404 RSP comes with built in integrated port adapter , that utilizes the chassis real estates to the fullest and provides customer dense 100/400 GbE ports inbuilt without additional MPA inserted to the chassis. 

- **Secure**: Apart from the Cisco's trusted IOS XR architecture, the 8404 RSP has in built crypto engine within the NPU , thus systme is available to support MACsec/IPSec on all varity of Ports and MPAs. 
  

## Cisco 8404 PIN(Place In Network) Use cases  

### Pre-aggregation & Aggregation

### Provider Edge Lite 
 
### IP and Optical Convergence    


## Platform Description  
### Chassis  

![8404-comp.png]({{site.baseurl}}/images/8404-comp.png)
Figure 3. 8404 Chassis & all Components
{: .text-center}  

As shown in Figure 3, above The 8404 Chassis comprise of the following Modular Components
- 2x Route Switch Processor(RSP)  slots
- 4x Modular Port Adpater slots
- 2x DC Power Entry Modules
- 1x Fan Tray containg 6 FANs

All of these Field replacable components are inserted into the chassis from front side and interconnected through chassis back plane

![8404-front-side.png]({{site.baseurl}}/images/8404-front-side.png)
Figure 4. Front to Side view of Cisco 8404 chassis
{: .text-center} 

![8404-back.png]({{site.baseurl}}/images/8404-back.png)
Figure 5. Back  view of Cisco 8404 chassis    
{: .text-center}  
  
### RSP (Route Switch Processor)  

The 8404-RSP1-48-EM RSP card is the 

![8404-RSP.png]({{site.baseurl}}/images/8404-RSP.png)
Figure 6. Cisco 8404 RSP Card: 8404-RSP1-48-EM
{: .text-center} 
### RSP Integrated Port Adpater (iPA)

### Modular Port Adapters (MPA)   
The Cisco 8404 router centralized platform supports two different types Modular Port Adapters(MPAs) that can be inserted into the four slots numbered 2 to 5. The MPA to slot mapping is any-to-any, means all MPA goes into any MPA slot in the system.    


**Cisco 8404  port adapters**  
The Cisco 8404 supported port adapters are summerized in the below table

| Port Adapter | Bandwidth | Port Combination| MACsec/IPSec | Timing |
| :---: | :---: | :---:| :---: | :---: |
|84-MPA-2H12Z-M| 800 G | 2x100G + 12x1/10/25/50 G | Yes | Class C |
|84-MPA-2FH/6H-M| 800 G | 2x400G or 6x100 G | Yes | Class C |



**84-MPA-2H12Z-M**   
This MPA is aimed at providing dense low speed(1/10/25) ports with a balanced 100G density. There are total 14 interface ports in the faceplate for a maximum total bandwidth of 800 Gbps that can support up to:  

- 12 ports of SFP56  modules and
	- Support 1/10/25/50GbE
- 2 ports of QSFP-28  modules
	- Support 40/100 GbE
    - Breakout Support  4x10/4x25GbE

![MPA2H12Z.png]({{site.baseurl}}/images/MPA2H12Z.png)
Figure 7. 84-MPA-2H12Z-M 
{: .text-center}  

**84-MPA-2FH/6H-M**  
This MPA is designed to provide dense 100G/400G density andcan be inserted into any slot (slots 0 to 7) of the chassis without any restriction. There are 6 QSFP cage in the faceplate to provide a maximum total bandwidth of 800 Gbps and it can support

Either, 
- 2 ports of QSFP-DD 400 GbE modules
	- Support 4x100G, 2x100G BO
Or,
- 6 Ports each used at 100G Ports
	- Support Breakout 4x10/4x25
The 6 front panel ports are organized in group of 3 ports (P0-P2 and P3-P5) that can individully support multiple combinations of speed and breakout options such as:
- 1x400G/4x100G + Disabled + Disabled
- 100G + 100G + 100G
- 4x25G + 100G + 100G
- 4x25G + 4x25G + Disabled
- 100G + 4x25G + Disabled
The above list is not exhaustive, detailed port speed and breakout options are discussed in later sections. 

![MPA2FH:6H.png]({{site.baseurl}}/images/MPA2FH:6H.png)

Figure 8. 86-MPA-4FH-M  
{: .text-center}  

### Power Entry Modules (PEMs)   

### Fan Trays (FTs)
  
## 8404 System Details   

### System Block Diagram  

### Route Switch Processor (RSP) to Port Adpater Mapping Details    

### 8404 Port Breakout Support

## 8404 Redundancy Details    

### Control & Data Plane Redunddnacy Architecture

### Fan redundancy

### PEM redundancy

## 8404 Software Details

### IOS XR Details
### MACsec Supprot
### Timing Supprot 

## Conclusion  
This document discussed the Cisco 8608 architecture.   
The Cisco 8608 is a unique platform that combines flexibility & reliability while offering investment protection. Customers can achieve unmatched reliability with redundant control and data plane via redundant Route Processors and Switch cards. Cisco 8608 Switch Cards are based on Cisco Silicon One™ Q200. A wide variety of Modular Port Adapters (MPAs) allow for high interface diversity.  

## Reference   
- [Cisco 8404 Datasheet](https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/8404-router-ds.html)  
- [Cisco 8404 Hardware Installation Guide]()        
- [Cisco 8000 Configuration guide](https://www.cisco.com/c/en/us/support/routers/8000-series-routers/products-installation-and-configuration-guides-list.html)        
- [Cisco Optics-to-Device Compatibility Matrix tool](https://tmgmatrix.cisco.com/)  
  

## Modification History  

| Version | Data     | Author(s)     | Comments            |
|---------|----------|---------------|---------------------|
| 1       | December-2025 | Paban Sarma | Initial Publication |

