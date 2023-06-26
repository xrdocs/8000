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

![figure2.png]({{site.baseurl}}/images/figure2.png){: .full}    
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


![]({{site.baseurl}}/images/figure3.png){: .full}  
Figure 3. Front view of an empty Cisco 8608 chassis  
{: .text-center}  

Cisco 8608 chassis utilizes a state-of-the-art orthogonal design with advanced cooling via front to back airflow, high power capacity and new power supplies. The chassis and all data path components.  

As shown in Figure 3, user needs to insert the followings:  
•	Modular Port Adapter(MPA) sits vertically in the front of the chassis
•	Route Processor(RP) sits vertically in the front of the chassis
•	Switch Card(SC) sits horizontally behind the Fan Trays. Cisco 8608 chassis has two Fan Trays with four Fan Modules on each Fan Tray
•	Midplane provides the control path connectivity and power distribution for the chassis



![figure4.png]({{site.baseurl}}/images/figure4.png){: .full}  
  Figure 4. Back view of Cisco 8608 chassis with 8 FMs and 4 PMs  
  {: .text-center}  
  In Figure 4, there are 2 Fan Trays with eight Fan Modules and four Power Modules  
  
  ![figure5.png]({{site.baseurl}}/images/figure5.png){: .full}    
  Figure 5. Side view of Cisco 8608 chassis  
  {: .text-center}  
  
  The Route Processors(RPs) are vertically inserted into the front right hand side of the Cisco 8608 chassis.
The Module Port Adapters(MPAs) are vertically inserted into the front side of the Cisco 8608 chassis too.
The Switch Cards(SC0 and SC1) that include the packet processing and forwarding engine using Q200 Cisco Silicon One are horizontally inserted into the back of the chassis.  There are 2 switch cards per system.  The Switch Cards contain the four removable Fan Modules can be inserted into each Switch Card.  


### MPA(Modular Port Adapter) Overview  
The Cisco 8608 Series centralized platform supports three different Redundant Modular Port Adaptors (MPAs) to insert into slot 0 to slot 7. 
This MPAs is CPU-less and have the PHY and Optics components for network connectivity to other devices in the network. Each Redundant MPA connects to both Switch Cards (SC0, SC1).  

![figure6.png]({{site.baseurl}}/images/figure6.png){: .align-center}  
Figure 6. two SCs and three Redundant MPAs connection of Cisco 8608 chassis  
{: .text-center}  
 
The Q200 on the SC has up to 32x 50Gbps SerDes to each MPA slot regardless of slot number.
There are three different Redundant MPAs in Cisco 8608.
The first MPA is the 86-MPA-14H2FH-M. This MPA can be inserted into any slot (slot 0 to slot 7) of the chassis without any restriction. This MPA is a pluggable card that provides 16 interface ports for a maximum total bandwidth of 1.6 Tbps that can support up to:
•	16 ports of QSFP 100 GbE modules (Default), or
•	2 ports of QSFP-DD 400 GbE modules and 14 ports of QSFP
When port 0 has a 400G QSFP-DD, then the other three ports (2, 3, 4) cannot be used. Similarly, when port 1 has a 400G QSFP-DD, then the other three ports (5, 6, 7) cannot be used.  

Note: No restriction in term of power or cooling for 400G ZR/ZR+/Bright ZR+    
{: .notice}  

![figure7.png]({{site.baseurl}}/images/figure7.png){: .full}    
Figure 7. 86-MPA-14H2FH-M  
{: .text-center}  

The second MPA is the 86-MPA-4FH-M. This MPA can be inserted into any slot(slot 0 to slot 7) of the chassis without any restriction. This MPA is a pluggable card that provides 4 interface ports for a maximum total bandwidth of 1.6 Tbps that can support up to:  
•	4 ports of QSFP-DD 400 GbE modules  

Note: No restriction in term of power or cooling for 400G ZR/ZR+/Bright ZR+    
{: .notice}  

![figure8.png]({{site.baseurl}}/images/figure8.png){: .full}   
Figure 8. 86-MPA-4FH-M  
{: .text-center}  

And third MPA is 86-MPA-24Z-M. This MPA can be inserted into any slot(slot 0 to slot 7) of the chassis without any restriction. This MPA is a pluggable card that provides 24 interface ports for a maximum total bandwidth of 1.2 Tbps that can support up to:
•	24 ports of SFP 10/25/50 GbE modules


![figure9.png]({{site.baseurl}}/images/figure9.png){: .full}    
Figure 9. 86-MPA-24Z-M  
{: .text-center}  

### RP Overview  
The RPs will vertically fit in the chassis from the front panel. There will be up to two redundant RPs in the system and will work in Active-standby mode. The RP is not connected to the Switch Card.  
RPs manages all routing operations on the Cisco 8600 Routers.  
![figure10.png]({{site.baseurl}}/images/figure10.png){: .full}  
Figure 10. 8608-RP  
{: .text-center}  

Front panel ports include management and Timing interfaces:  
•	RJ45 console port directly connected to CPU  
•	10/100/1000 Mbps Management Ethernet Port  
•	USB 3.0 (Type A)  
•	Timing ports: 1588 PTP SFP, SyncE, BITS, GNSS PRTC-A, and GPS(ToD with RJ45 port, 1PPS and 10MHz coax input)  
RP contains an Intel 6-core @ 2.5 GHz, 64GB DDR4 DRAM, and 240 GB M.2 SSD Flash Drive
This RPs are connected to midplane and communicate with their devices via PCIE, EOBC, P2PM, and I2C Interfaces.  

### Switch Card Overview  
The Cisco 8608 Series Switch Card sits behind the Fan Trays. Cisco 8608 chassis has two Fan Trays with total eight Fan Modules. The Cisco Silicon One Q200 NPU sits within the Switch Card and all the packet process and forwarding decisions are driven by the Switch Card.  

![figure11.png]({{site.baseurl}}/images/figure11.png){: .full}  
Figure 11. 8608-SC0-128  
{: .text-center}  

Cisco 8608 can operate with up to 2 Switch Cards(SCs) between MPA and Fan Trays.  

![figure12.png]({{site.baseurl}}/images/figure12.png){: .full}  
Figure 12. Redundant SCs in the back of the Cisco 8608  
{: .text-center}  

The RP & SC operate in pairs RP0-SC0 and RP1-SC1 to deliver redundancy without loss of any chassis front face plate.
The two SCs are CPU-less connected to both the RPs via PCIE, P2PM, EOBC links and each SC has one Cisco Silicon One Q200 NPU providing the data path for packet processing and forwarding.  

![figure13.png]({{site.baseurl}}/images/figure13.png){: .full} 
Figure 13. Redundant System  
{: .text-center}  

The SC is installed in the rear of the chassis. The 8 fan trays are installed into the SCs. If you need to remove or replace the SC, we recommend that you remove the Fan Trays.
To avoid over-heating the system, the service time between when a switch card is removed and re-inserted must be under 5 minutes at 30C. Individual Fan Module may be removed with no time limit.
If there is only one SC card plugged in, a Fan Spinner card will be required which is essentially a SC blank slot with a Fan Controller and Fan Modules.

![figure14.png]({{site.baseurl}}/images/figure14.png){: .full} 
Figure 14. Non-Redundant System  
{: .text-center}  


Fan Spinner does not provide any data or control plane functionality.  It is a passive card used to connect the midplane to the Fans.  

### Power Modules Overview  

You can install up to four 3200W AC or 3200W DC power supplies in the chassis. Ensure that all power connection wiring conforms that to the rules and regulations in the National Electrical Code (NEC) and in local codes.  

Note: The Cisco 8608 chassis doesn't support a mix of AC and DC Power Supply Units (PSUs)  
{: .notice}  

AC Power Module (86-3.2KW-AC)  
• The AC power modules are single feed with 3200W capacity at 220V. They also support operation at 110V with half the capacity.  
• AC power modules are rated at 3200KW maximum at 230VAC high line input, and 1570W maximum at 115VAC low line input.  
• AC nominal range: 100—1230VAC and 2002—40VAC  
• AC full range: 85—132 VAC and 180—264VAC; with extended range to 300VAC  

DC Power Module (86-3.2KW-DC)  
• The AC power modules are single feed with 3200W capacity at 220V. They also support operation at 110V with half the capacity.  
• AC power modules are rated at 3200KW maximum at 230VAC high line input, and 1570W maximum at 115VAC low line input.  
• AC nominal range: 100—1230VAC and 2002—40VAC  
• AC full range: 85—132 VAC and 180—264VAC; with extended range to 300VAC  
The airflow direction through the Fan Trays and Power Modules on the Cisco 8608 routers are from front to back (port side intake).  
The maximum power available for operations depends on the input power from your power source, the number and output capabilities of power supplies, and the power redundancy mode that you use.   
The following table lists the amount of power available for Cisco 8600 series routers from all available power trays.   

![powersupply.png]({{site.baseurl}}/images/powersupply.png){: .full} 


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco8608#sh plat
0/RP0/CPU0        8608-RP(Active)          IOS XR RUN               NSHUT
0/RP1/CPU0        8608-RP(Standby)         IOS XR RUN               NSHUT
0/SC0             8608-SC0-128             OPERATIONAL              NSHUT
0/SC1             8608-SC0-128             OPERATIONAL              NSHUT
0/FB0             8608-SC0-128[FB]         OPERATIONAL              NSHUT
0/FB1             8608-SC0-128[FB]         OPERATIONAL              NSHUT
0/0               86-MPA-4FH-M             OPERATIONAL              NSHUT
0/4               86-MPA-14H2FH-M          OPERATIONAL              NSHUT
0/5               86-MPA-24Z-M             OPERATIONAL              NSHUT
0/6               86-MPA-14H2FH-M          OPERATIONAL              NSHUT
0/7               86-MPA-14H2FH-M          OPERATIONAL              NSHUT
0/FT0             8608-FAN                 OPERATIONAL              NSHUT
0/FT1             8608-FAN                 OPERATIONAL              NSHUT
0/FT2             8608-FAN                 OPERATIONAL              NSHUT
0/FT3             8608-FAN                 OPERATIONAL              NSHUT
0/FT4             8608-FAN                 OPERATIONAL              NSHUT
0/FT5             8608-FAN                 OPERATIONAL              NSHUT
0/FT6             8608-FAN                 OPERATIONAL              NSHUT
0/FT7             8608-FAN                 OPERATIONAL              NSHUT
0/PM0             PSU3.2KW-ACPI            OPERATIONAL              NSHUT
0/PM1             PSU3.2KW-ACPI            OPERATIONAL              NSHUT  
</code>
</pre>
</div>  

On this router, there are two Power Modules in Cisco 8608 chassis.  


<div class="highlighter-rouge">
<pre class="highlight">
<code>  
RP/0/RP0/CPU0:Cisco8608#sh environment power
================================================================================
CHASSIS LEVEL POWER INFO: 0
================================================================================
   Total output power capacity (N + 1)             :    6400W +        0W
   Total output power required                     :    4465W
   Total power input                               :    1122W
   Total power output                              :    1028W
================================================================================
   Power       Supply         ------Input----   ------Output---     Status
   Module      Type            Volts     Amps    Volts     Amps    
================================================================================
   0/PM0       PSU3.2KW-ACPI   207.9     2.8     54.7      9.8      OK
   0/PM1       PSU3.2KW-ACPI   207.9     2.6     54.7      9.0      OK
Total of Power Modules:       1122W/5.4A              1028W/18.8A
================================================================================
   Location     Card Type               Power       Power        Status
                                        Allocated   Used
                                        Watts       Watts
================================================================================
   0/RP0/CPU0   8608-RP                 200         53           ON
   0/RP1/CPU0   8608-RP                 200         52           ON
   0/SC0        8608-SC0-128            542         164          ON
   0/SC1        8608-SC0-128            542         165          ON
   0/FB0        8608-SC0-128[FB]        10          -            ON
   0/FB1        8608-SC0-128[FB]        10          -            ON
   0/0          86-MPA-4FH-M            209         115          ON
   0/1          -                       32          -            RESERVED
   0/2          -                       32          -            RESERVED
   0/3          -                       32          -            RESERVED
   0/4          86-MPA-14H2FH-M         290         156          ON
   0/5          86-MPA-24Z-M            198         77           ON
   0/6          86-MPA-14H2FH-M         284         149          ON
   0/7          86-MPA-14H2FH-M         284         147          ON
   0/FT0        8608-FAN                200         12           ON
   0/FT1        8608-FAN                200         12           ON
   0/FT2        8608-FAN                200         12           ON
   0/FT3        8608-FAN                200         12           ON
   0/FT4        8608-FAN                200         15           ON
   0/FT5        8608-FAN                200         15           ON
   0/FT6        8608-FAN                200         15           ON
   0/FT7        8608-FAN                200         15           ON  
</code>
</pre>
</div>  


<div class="highlighter-rouge">
<pre class="highlight">
<code>  
RP/0/RP0/CPU0:Cisco8608#sh environment fan
=============================================================
                                              Fan speed (rpm)
Location      FRU Type                        FAN_0    FAN_1   
-------------------------------------------------------------
0/FT0        8608-FAN                           3840    3840   
0/FT1        8608-FAN                           3840    3840   
0/FT2        8608-FAN                           3840    3810   
0/FT3        8608-FAN                           3870    3810   
0/FT4        8608-FAN                           3930    3870   
0/FT5        8608-FAN                           3870    3930   
0/FT6        8608-FAN                           3900    3870   
0/FT7        8608-FAN                           3870    3900   
0/PM0        PSU3.2KW-ACPI                      5225    5247   
0/PM1        PSU3.2KW-ACPI                      5204    5247  
</code>
</pre>
</div>  

Inside the chassis, there are two Route Processors (RPs), two Switch Cards (SCs), eight Modular Port Adapters (MPAs), 4 Power Modules (PMs), 8 Fan Modules (FMs) – all these entities are field replaceable units (FRUs).  

### IOS XR Software  
Cisco IOS XR 7.10.1 introduces the Cisco 8608 router support.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>  
RP/0/RP0/CPU0:Cisco8608#sh version

Cisco IOS XR Software, Version 7.10.1.25I LNT
Copyright (c) 2013-2023 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Thu May 11 23:47:35 UTC 2023
 Build Host   : iox-ucs-058
 Workspace    : /auto/iox-ucs-058-san2/prod/7.10.1.25I.SIT_IMAGE/8000/ws/
 Version      : <mark>7.10.1.25I</mark>
 Label        : 7.10.1.25I

cisco 8000 (Intel(R) Xeon(R) CPU D-1633N @ 2.50GHz)
cisco 8608-SYS (Intel(R) Xeon(R) CPU D-1633N @ 2.50GHz) processor with 64GB of memory
ios uptime is 2 days, 20 hours, 30 minutes
Cisco 8600 - 8 Slot Centralized Chassis  
</code>
</pre>
</div>  

Detailed information on XR7 can be found here: [IOS XR Data Sheet](https://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/ios-xr-software/datasheet-c78-743014.html).
For a complete list of supported features, refer to the [Cisco Feature Navigator](https://tools.cisco.com/ITDIT/CFN/jsp/index.jsp).
[Cisco Software Download](https://software.cisco.com/download/home/286325065/type/280805694/release/7.3.1) page to download the Cisco IOS XR software images.




## MPA

## Redundancy

## Conclusion
