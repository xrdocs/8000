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

  
The key characteristics of the Cisco 8712-MOD-M are summarized below.     

![product-specs.png]({{site.baseurl}}/images/product-specs.png)

## Product Video

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=2ckU9oAP9hI&t=17s" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

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
- Route Reflector role

Cisco 8712-MOD-M is also powered to address some of the enterrpise and data center/Cloud segments:
- Enterprise WAN core & service Edge
- Enterprise Internet Gateway (Peering)
- Enterprise CPE
- Enterprise IP Backbone (Core)
- Data Center Gateway (DC-GW) 
- Data Center Interconnect
- Cloud Interconnect
    

# 8712-MOD-M System Network Processing Unit(NPU) Architecture
Cisco 8712-MOD-M system is powered by Cisco Silicon One™ K100 Network Processing Unit(NPU), a first in class to deliver intelligent Agregation/Edge services. Get a kick start with: [Cisco-8000-architecture](https://xrdocs.io/8000/blogs/Cisco-8000-QOS-architecture/) to understand the fundamentals of Cisco Silicon One™ Network Processing Unit (NPU). 

![k100.png]({{site.baseurl}}/images/k100.png){: .align-center}

Figure 1. Cisco Silicon One K100 NPU building blocks{: .text-center}

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


# 8712-MOD-M System Internal Block Diagram

Let us check how 8712-MOD-M system is architected with K100 Silicon One™ NPU,

![sys-blocks.png]({{site.baseurl}}/images/sys-blocks.png){: .align-center}

- 2 network slices with 2 IFGs per slice 
- Each IFG has 32x 56G raw bandwidth throughput
- Each IFG hosts one MPA with a throughput capacity of 1.6Tbps
- Each MPA is serviced by one IFG
- Any variant of MPA can be plugged into any of the 4 bays

# 8712-MOD-M Chassis design
Cisco 8712-MOD-M is a Cisco Silicon One™ based 2RU fixed router belonging to Cisco 8700 family of products, a first in class device to play multiple roles across Service Provider, Enterprise WAN , Data Center Interconnect, DC gateway network applications.

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

## Front Panel Management and Timing Provisions
Let us have a telescopic view of the front panel deatils,

![fron-panel-mgmnt-ports.png]({{site.baseurl}}/images/fron-panel-mgmnt-ports.png){: .align-center}
Figure 3. Front panel ports of Cisco 8712-MOD-M  
{: .text-center} 

Front panel ports include management and timing interfaces:

- Management interfaces  
  - 1x 10/100/1000 Mbps Management Ethernet port  
  - 1x USB 3.0  
  - RJ45 console port directly to CPU  
  - 1x 10 GbE Control plane Expansion Ethernet port (Not used)  
- Timing interfaces  
8712-MOD-M is capable of frequency, time, and phase synchronization. These can be sourced from dedicated GPS port or from timestamped 1588 packets received on a normal data port in the system. The time, phase and frequency are distributed through the system with a frequency and PPS signal to devices near the physical port where timestamps are applied on ingress and egress packets.  
In 8712-MOD-M, network synchronization clock can be sourced from any of the ports on any of the MPAs and from GPS (ToD, 10MHz, 1PPS ports).  
- 1x GPS interface  
  - ToD (Time of Day) with RJ45 port  
  - 1 PPS coax port  
  - 10 MHz coax port  
- 1x GNSS (Global Navigation Satellite System) receiver

# Modular Port Adaptors (MPAs)
Cisco 8712-MOD-M system has 4 bays to provision MPAs. There are 4 MPA variants supported on this system and any combination of MPA varinats can be plugged into the MPA 4 slots. And these MPAs are Online Insertion & Removal capable and user has the flexibility to choose the MPA varinats and number of MPAs based on the capacity provisioning of the network.

![mpa-bay-view.png]({{site.baseurl}}/images/mpa-bay-view.png)
Figure 4. MPA bay view of Cisco 8712-MOD-M
{: .text-center}


## 8K-MPA-4D

This MPA can power high bandwidth uplink capacity provisioning and also does its best with digital coherent optics (DCO) in Routed Optical networking. All ports supports inline MACSec and Class C timing on native ports.

![4d.png]({{site.baseurl}}/images/4d.png)

![4d-a.png]({{site.baseurl}}/images/4d-a.png)

- This MPA has four QSFPDD ports
- All four ports can be operated in breakout mode
- All four ports supports 4x10G or 4x25G or 4x100G breakout mode
- All four ports can be populated with DCO optics

|     Speed     | 40 GbE  | 400     |
|---------------|---------|---------|
| Native Ports  | 4       | 4       |


### Breakout Port configurations through Retimer / Gear box 

This MPA has two Retimer/Gearbox and each has 16x 50Gbps serdes lines towards the NPU side which facilitate it to support 16 individual port configurations on it.

|      Speed     | 10 GbE  | 25 GbE | 100 GbE |
|----------------|---------|--------|---------|
| Breakout Ports | 16      | 16     | 16      |


### How to enable Breakout
Default port mode of all 4 ports is 400G. So no need to change the port mode before breakout configuration

- Configure the optical breakout 4x10 or 4x25 or 4x100

Below is 4x10g breakout configuration example captured on the system,

![4d-bo-cfg.png]({{site.baseurl}}/images/4d-bo-cfg.png){: .align-center}


## 8K-MPA-16H
This MPA suites for dense 100G fan outs and also supports 4x10G or 4x25G breakout configurations
All ports supports inline MACSec and Class C timing on native ports.

![16h-screen.png]({{site.baseurl}}/images/16h-screen.png){: .align-center}


![16h-a.png]({{site.baseurl}}/images/16h-a.png){: .align-center}


|     Speed     | 40 GbE  | 100     |
|---------------|---------|---------|
| Native Ports  | 8       | 16      |

- 4x 10G / 4x 25G breakout is supported on even numbered ports: 0,2,4,6,8,10,12,14 
- Odd numbered ports get disabled after even numbered ports are configured with breakout

### Breakout Port configurations through Retimer / Gear box 

This MPA has four Retimer/Gearbox modules and each has 8x 50Gbps serdes lines towards the NPU side which limits maximum number of individual port configuration on it to 8 as each port need atleast one serdes line. So enabling breakout configuration on even numbered port will disable the immediate odd numbered port. ie, breakout configuration on port-4 will disable port-5.


|     Speed      | 10 GbE  | 25 GbE |
|----------------|---------|--------|
| Breakout Ports | 32      | 32     |

### How to enable Breakout

Change port mode to 40 as default port mode of port 0 - 15 is 100G.

- Configure the port intended to breakout in mode 40
- Configure the optical breakout 4x10 or 4x25

Below is 4x10g breakout configuration example captured on the system,

![16h-b0-example.png]({{site.baseurl}}/images/16h-b0-example.png)



## 8K-MPA-16Z4D
This MPA suites for deployments having combination of low speed and high speed port connectivity.
It has 16x SFP56 low speed ports which supports 1GbE, 10GbE, 25GbE & 50GbE. And it also has 4x QSFPDD high speed connectivity which can support 4x 100GbE or 2x 400GbE. All ports supports inline MACSec and Class C timing only on QSFPDD ports.

Default port speed of low speed ports (Ports 0 - 15) is 50GbE and 100GbE for high speed ports (Port 16 - 19)

![16z-screen.png]({{site.baseurl}}/images/16z-screen.png){: .align-center}

![16z-arch.png]({{site.baseurl}}/images/16z-arch.png){: .align-center}


|  Speed   | 1 GbE   | 10 GbE  | 25 GbE | 40 GbE  | 50 GbE  | 100     | 400 GbE |
|----------|---------|---------|--------|---------|---------|---------|---------|
| Native   | 16      | 16      | 16     | 2       | 16      | 4       | 2       |


- 4x 10G / 4x 25G / 4x 100G breakout is supported on port-16 & 18
- Odd numbered ports get disabled after even numbered ports are configured with breakout

### Breakout Port configurations through Retimer / Gear box 

This MPA has three Retimer/Gearbox modules where 2 are hosting QSFP-DD form factor optics and each has 8x 50Gbps serdes lines towards the NPU side. But breakout is supported only on 2 QSFDD ports (Port-16 & 18) out 4 ports (Port16-19).So enabling breakout configuration on even numbered port will disable the immediate odd numbered port. ie, breakout configuration on port-16 will disable port-17.


|     Speed      | 10 GbE  | 25 GbE | 100 GbE |
|----------------|---------|--------|---------|
| Breakout Ports | 8       | 8      | 8       |


### How to enable Breakout
Default port mode of Port 16 - 19 is 100G. So no need to change the port mode before breakout configuration

- Configure the port intended to breakout in mode 400
- Configure the optical breakout 4x10 or 4x25 or 4x100

Below is 4x100g & 4x10g breakout configuration example captured on the system,

![combo-100BO.png]({{site.baseurl}}/images/combo-100BO.png){: .align-center}

![10g-bo-combo.png]({{site.baseurl}}/images/10g-bo-combo.png){: .align-center}



## 8K-MPA-18Z1D
This MPA is similar to 8K-MPA-16Z2D but it has 18x SFP56 low speed ports which supports 1GbE, 10GbE, 25GbE & 50GbE and 1x QSFPDD high speed connectivity which can support one 100GbE or 400GbE optics. All ports supports inline MACSec and Class C timing.

Default port speed of low speed ports (Ports 0 - 7 & 9 - 18) is 50GbE and 400GbE for high speed port(Port 8)

![lotus-new.png]({{site.baseurl}}/images/lotus-new.png){: .align-center}




![18z-arch.png]({{site.baseurl}}/images/18z-arch.png){: .align-center}



|  Speed   | 1 GbE   | 10 GbE  | 25 GbE | 40 GbE  | 50 GbE  | 100     | 400 GbE |
|----------|---------|---------|--------|---------|---------|---------|---------|
| Native   | 18      | 18      | 18     | 1       | 18      | 1       | 1       |


- 4x 10G / 4x 25G / 4x 100G breakout is supported on port-8

### Breakout Port configurations through Retimer / Gear box 

This MPA has 2 PHY modules and 4x10 or 4x25 or 4x100 breakout configuration is supported only on Port-8 without any restriction.


|     Speed      | 10 GbE  | 25 GbE | 100 GbE |
|----------------|---------|--------|---------|
| Breakout Ports | 4       | 4      | 4       |


### How to enable Breakout
Default port mode of port-8 is 400G. So no need to change the port mode before breakout configuration

- Configure the optical breakout 4x10 or 4x25 or 4x100

Below is 4x100g & 4x10g breakout configuration example captured on the system,
![lotus-bo-1.png]({{site.baseurl}}/images/lotus-bo-1.png){: .align-center}

![lotus-bo-2.png]({{site.baseurl}}/images/lotus-bo-2.png){: .align-center}


## Maximum Port scale possible in a fully loaded System

The following table represents the maximum number of ports the Cisco 8712-MOD-M router can support.  

|  Speed   | 1 GbE   | 10 GbE  | 25 GbE | 40 GbE  | 50 GbE  | 100     | 400 GbE |
|----------|---------|---------|--------|---------|---------|---------|---------|
| Native   | 72      | 72      | 24     | 32      | 72      | 64      | 16      |
| Breakout | NA      | 128     | 128    | NA      | NA      | 64      | NA      |

**Notes** : Maximum number of ports mentioned in each categories in above table is by considering that the system is loaded with maximum number of respective MPA variants. Like, system has to be loaded with 4x 8K-MPA-18Z1D to get 72 ports of 1GbE or 10GbE or 25GbE or 50GbE port configuration.

Similarly, system can get a configuration of 128 ports of 10GbE or 25GbE by using four 8K-MPA-16H  MPA variant with 8 ports are configured with 4x 10GbE or 25GbE breakout on all 4 MPAs.

## 8712-MOD-M system Interface Nomenclature
Let us look at the interface naming criteria for this system. This system is categoriese under fixed form factor with a front panel modularity. Because of its modular nature each Modular Port Adaptor (MPA) bay need to be numbered as per the slot number and correcponsing ports will get numbered rooted from the slot number accordingly. Nomenclature is detailed as below,

![nomen-latest.png]({{site.baseurl}}/images/nomen-latest.png){: .align-center}


### Understand the interface naming from below example
Below output is captured from 8712-MOD-M system and system is loaded with all 4 types of MPAs in this case. MPA slots are identified as 0/0, 0/1, 0/2, 0/3 as seen below.

![show-plat.png]({{site.baseurl}}/images/show-plat.png)

Below output helps to understand the interface naming briefed in the nomenclature section above. Interfaces are colored according to the color used for each MPA slots in above output. And this output also helps to understnad the IFG/Slice mapping of each interfaces and the VOQ IDs of each.

![port-table.png]({{site.baseurl}}/images/port-table.png)


*Understand Cisco 8712-MOD-M’s modules and associated PIDs

| PIDs             | Product Description                                           |
|------------------|---------------------------------------------------------------|
| 8712-MOD-M       | Cisco 8712 2RU 6.4T KR100 System with 4 MPA bays              |
| 8K-MPA-16H       | Cisco 8000 16 x 100G MPA                                      |
| 8K-MPA-4D        | Cisco 8000 4 x 400G MPA                                       |
| 8K-MPA-16Z2D     | Cisco 8000 16x50G + 2x400G MPA                                |
| 8K-MPA-18Z1D     | Cisco 8000 18x50G + 1x400G MPA                                |
| FAN-PI-V3        | Cisco 8000 FAN - port-side intake                             |
| FAN-PE-V3        | Cisco 8000 FAN - port-side Exhaust                            |
| PSU2KW-ACPI      | 2KW AC Power Module with Port-side Air Intake                 |
| PSU2KW-ACPE      | 2KW AC Power Module with Port-side Air Exhaust                |
| PSU2KW-DCPI      | 2KW 48V DC Power Module with Port-side Air Intake             |
| PSU2KW-DCPE      | 2KW 48V DC Power Module with Port-side Air Exhaust            |
| PSU2KW-HVPI      | 2KW AC/HVAC/HVDC Power Supply with Port-side Air Intake       |


# Conclusion  

This document discussed the Cisco 8712-MOD-M architecture and its use cases. Cisco 8712.MOD-M is a Cisco Silicon One™ K100 based 2RU fixed routing platform with front panel port modularity. Modularity is achieved by 4 different types of Modular Port Adaptors(MPAs) which offers diverse port speed : 1G, 10G, 25G, 40G, 50G, 100G, and 400G. 8712-MOD-M system addresses wide range of use cases which are detailed in above sections.

# References  
[Cisco 8000 Cisco Silicon One™ Architecture & QOS](https://xrdocs.io/8000/blogs/Cisco-8000-QOS-architecture/)      
[Cisco 8712-MOD-M Datasheet](https://www.cisco.com/c/en/us/products/se/2024/8/Collateral/8700-series-routers-ds.html#8712MODMrouter)                 
[Cisco 8712-MOD-M Hardware Installation Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/hardware/hig-fixed-8700/b-8700-hardware-installation-guide/routers_overview.html)       
[Cisco 8000 Configuration guide](https://www.cisco.com/c/en/us/support/routers/8000-series-routers/products-installation-and-configuration-guides-list.html)           
[Cisco Optics-to-Device Compatibility Matrix tool](https://tmgmatrix.cisco.com/)    
[Cisco 8000 Power Calculator](https://8000-power.cisco.com/)    


# Modification History  

| Version | Data     | Author(s)     | Comments            |
|---------|----------|---------------|---------------------|
| 1       | Dec, 2025 | Ram Mohan A M | Initial Publication |
