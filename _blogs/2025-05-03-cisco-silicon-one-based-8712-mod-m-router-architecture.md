---
published: false
date: '2025-05-03 12:26 +0530'
title: Cisco Silicon One™ based 8712-MOD-M Router Architecture
---
|Ram Mohan A.M., Technical Leader, Technical Marketing Engineering (raam@cisco.com)|  
{% include toc icon="table" title="Cisco Silicon One™ based 8712-MOD-M Router Architecture White Paper" %}



# Introduction
The Cisco 8000 Series combines the revolutionary Cisco Silicon One™, IOS XR® software, and a set of clean sheet chassis to deliver a breakthrough in high-performance routers. 
High-performance networking systems have historically been divided into routing or switching classes, with distinct hardware and software. But, as the necessity for reliable networks proliferates, it is imperative that traditional networks shift to a new architecture to address exponential bandwidth needs, ubiquitous connectivity, security, better quality of service, low latency coupled with high reliability, availability, and serviceability of the infrastructure. The Cisco® 8000 Series portfolio completes this journey with the Cisco 8700 products.
The Cisco 8700 series is an addition to [the Cisco 8000 Series Routers](https://www.cisco.com/site/us/en/products/networking/routers/8000-series/index.html) powered by [Cisco Silicon One](https://www.cisco.com/site/us/en/products/networking/silicon-one/index.html)™  ASICs.  8700 products embody this cutting-edge innovation of the 8000 portfolio, offering advanced features engineered for seamless integration and scalability. Whether enhancing existing infrastructure or enabling new capabilities, the 8700 series products empower organizations to achieve their goals with unmatched efficiency and effectiveness. 

# Cisco 8712-MOD-M Overview  
Cisco 8712-MOD-M is a 2RU 6.4Tbps fixed system based on a single Cisco Silicon One™ K100 Network Processing Unit (NPU). This system offers 4 bays of Modular Port Adaptors (MPAs) with each slot having 1.6Tbps capacity totalling 6.4Tbps at system level. 4 types of MPAs are supported which offers diverse port speed combinations supported by SFP56, QSF28, QSFPDD optical form factors: 1G, 10G, 25G, 50G, 100G, 400G.

## Product Video  

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=2ckU9oAP9hI&t=17s" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>  
  
The key features of the Cisco 8712-MOD-M are summarized below.     

![product-specs.png]({{site.baseurl}}/images/product-specs.png)

## Service Provider(SP) networking Architecture shift


Lets look at traditional SP network architecture and the how it is transitioning to nextGen network driven by Agile Services Networking architecture.

Network roles are strictly defined and deployed in networks. Devices must be groomed specifically to address different roles in network. And feature capability , scale and bandwidth requirements are differently fitted to address different needs in the network.

Common view of traditional service provider network is as following, different services like business, wireless, mobile etc. are terminated into the transport aggregation which includes further nailing down to access, preaggregation and aggregation network segments etc.. Aggregated transport networks talk to Edge then merge into the  Core segment. Edge can segregated to cater to different use cases as per the specific need of each categories like, Business edge, Mobile edge, Wireless edge etc..or all types of edge services can be centrally placed and provisioned.


# Cisco 8712-MOD-M Use Cases  
As user applications become more dynamic network is demanding differentiated user experiences, assurance of low latency and high availability without any compromise in keeping the real estate and energy in green at its best promising levels. And Cisco's Agile Services Networking (ASN)architecture is shaping these demands well by simplifying the network by unifying the network capabilities across different network segment islands into single architecture.
The Cisco 8712-MOD-M powered by Cisco Silicon One™ K100 NPU is a well architected platform to empower ASN architecture to cater diversified deployment usecases across various network domains and segments. The highly flexible front panel provisioniong capability makes it very versatile in deployments: high speed bandwidth pipe to towards upstream peering , low speed service aggregation towards downstream peering, dense 100G aggregation and out-of-band route reflector role where system can be provisioned with minimal port connectivity by keeping only single MPA bay loaded and rest filled with MPA blanks and so on... Cisco 8712-MOD-M also looking at other networking segments, securing the connections of multiple data centers and cloud environments, WAN MACsec sometimes called data center interconnect (DCI), backhauling remote branch sites is critical to the success of many commercial, enterprise, and government organizations. In the consumer space, backhauling is important for remote stores and point-of-sale kiosks. In the government space, it is crucial for the operation of remote agencies and offices.  

Detailed list of usecases are categoriesed as below,

Cisco 8712-MOD-M is powered to address below roles in service providers:
- Internet gateway and exchange (Peering) roles
- CDN Peering role (SP DC)
- Datacenter Interconnect Gateway role
- Metro edge and aggregation service role
- Small & Medium Mobile service PE role
- Enterprise Business to Business service PE role
- Routed Optical Netwroking role
- Heavy Access termination role
- Route Reflector

Cisco 8712-MOD-M is also powered to address some of the enterrpise and data center/Cloud segments:
- Enterprise WAN service Edge
- Enterprise Internet Gateway (Peering)
- Enterprise CPE
- Enterprise IP Backbone (Core)
- Data Center Gateway (DC-GW) 
- Data Center Interconnect
- Cloud Interconnect
    

# System Architecture
Cisco 8712-MOD-M system is powered by Cisco Silicon One™ K100 Network Processing Unit(NPU), a first in class to deliver intelligent Edge services. Get a kick start with: [Cisco-8000-architecture](https://xrdocs.io/8000/blogs/Cisco-8000-QOS-architecture/)To understand the fundamentals of Cisco Silicon One™. 



![k100-blocks.png]({{site.baseurl}}/images/k100-blocks.png){: .align-center}    
Figure 14. Cisco Silicon One K100 Details  
{: .text-center}



Cisco Silicon One™ K100 NPU features:
- 6.4Tbps full duplex forwarding capacity @2.6Bpps
	- has 2 network slices
	- each slice has a forwarding capacity of 3.2Tbps
	- has 128x56G SerDes
	- 2 Interfaec Groups (IFGs) per slice
    - has 64MB on-chip packet memory
	- has 8GB HBM deep buffering memory co-packaged with NPU
- On-chip crypto engine for MACsec & IPsec applications
- Multiple embedded ARC processors for CPU offloading
- 128K VoQ (Virtual Output Queue)
- 1M counters to facilitate statistics punching for different feature applications and built-in counters in HCAM(Hash-based TCAM)
- 6M IPv4 or 4M IPv6 FIB scaling capacity


## 8712-MOD-M System Block Diagram

Let us check how 8712-MOD-M system is architected with K100 Silicon One™ NPU,

![sys-blocks.png]({{site.baseurl}}/images/sys-blocks.png){: .align-center}
{: .text-center}

- 2 network slices with 2 IFGs per slice 
- Each IFG has 32x 56G raw bandwidth throughput
- Each IFG hosts one MPA with a throughput capacity of 1.6Tbps
- Each MPA is serviced by one IFG
- Any variant of MPA can be plugged into any of the 4 bays

## Platform Description
Cisco 8712-MOD-M is a Cisco Silicon One™ based 2RU fixed router belonging to Cisco 8700 family of products, a first in class device to play intelligent Edge roles across Service Provider, Enterprise WAN , Data Center Interconnect, DC gateway network applications.

![fron-view.png]({{site.baseurl}}/images/fron-view.png)
Figure 1. Front view of the Cisco 8712-MOD-M  
{: .text-center}  

MPA bays can be populated with any combination of MPA types

![fron-rear-full-chassi-view.png]({{site.baseurl}}/images/fron-rear-full-chassi-view.png)
Figure 2. Front & Rear view of Cisco 8712-MOD-M  
{: .text-center} 

8712-MOD-M system is powered by,

- Two 2KW Power Distribution Units
- Supports both AC & DC power units
- Separate power unit models for Port-side air intake(PI) & Port-side air exhaust(PE) air flows
- Systems supprots 1+1 Redundancy

3+1 Redundancy
Separate PID for PI & PE Air flows
FAN-PI-V3 / FAN-PE-V3



![fron-panel-mgmnt-ports.png]({{site.baseurl}}/images/fron-panel-mgmnt-ports.png)
Figure 3. Front panel ports of Cisco 8712-MOD-M  
{: .text-center} 



# Modular Port Adaptors (MPAs)
Cisco 8712-MOD-M system has 4 bays to provision MPAs. There are 4 MPA variants supported on this system and any combination of MPA varinats can be plugged into the MPA 4 slots. And these MPAs are Online Insertion & Removal capable and user has the flexibility to choose the MPA varinats and number of MPAs based on the capacity provisioning of the network.

![mpa-bay-view.png]({{site.baseurl}}/images/mpa-bay-view.png){: .full}
Figure 4. MPA bay view of Cisco 8712-MOD-M  
{: .text-center}


## 8K-MPA-4D

This MPA can power high bandwidth uplink capacity provisioning and also does its best with digital coherent optics (DCO) in Routed Optical networking.

![4d.png]({{site.baseurl}}/images/4d.png){: .align-center}

![4d-a.png]({{site.baseurl}}/images/4d-a.png)
{: .align-center}

- This MPA host four QSFPDD ports
- All four ports can be operated in breakout mode
- All four ports supports 4x10G or 4x25G or 4x100G breakout mode
- All four ports can be populated with DCO optics

## 8K-MPA-16H
![16h-screen.png]({{site.baseurl}}/images/16h-screen.png){: .align-center}


![16h-a.png]({{site.baseurl}}/images/16h-a.png){: .align-center}


- This MPA serves dense 100G fan outs
- 4x 10G / 4x 25G breakout is supported on even numbered ports: 0,2,4,6,8,10,12,14 
- Odd numbered ports get disabled after even numbered ports are configured with breakout



## 8K-MPA-16Z4D

![16z-screen.png]({{site.baseurl}}/images/16z-screen.png){: .align-center}

![16z-arch.png]({{site.baseurl}}/images/16z-arch.png)



## 8K-MPA-18Z1D

![18z-arch.png]({{site.baseurl}}/images/18z-arch.png)
{: .align-center}







*Understand Cisco 8712-MOD-M’s modules and associated PIDs

| PIDs             | Product Description                                           |
|------------------|---------------------------------------------------------------|
| 8712-MOD-M       | Cisco 8712 2RU 6.4T KR100 System with 4 MPA bays              |
| 8K-MPA-16H       | Cisco 8000 16 x 100G MPA                                      |
| 8K-MPA-4D        | Cisco 8000 4 x 400G MPA                                       |
| 8K-MPA-16Z2D     | Cisco 8000 16x50G + 2x400G MPA                                |
| FAN-PI-V3        | Cisco 8000 FAN - port-side intake                             |
| FAN-PE-V3        | Cisco 8000 FAN - port-side Exhaust                            |
| PSU2KW-ACPI      | 2KW AC Power Module with Port-side Air Intake                 |
| PSU2KW-ACPE      | 2KW AC Power Module with Port-side Air Exhaust                |
| PSU2KW-DCPI      | 2KW 48V DC Power Module with Port-side Air Intake             |
| PSU2KW-DCPE      | 2KW 48V DC Power Module with Port-side Air Exhaust            |
| PSU2KW-HVPI      | 2KW AC/HVAC/HVDC Power Supply with Port-side Air Intake       |


