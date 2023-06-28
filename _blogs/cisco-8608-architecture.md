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
- Chassis: 7RU modular chassis which hosts commons (fans, power supplies), Route Processors (RP), Switch Cards (SC) and Modular Port Adapters (MPAs).  
- Route Processor (RP): Cisco 8608 RP hosts the CPU and the I/O ports on it. RPs fit vertical in the chassis and are inserted from the front panel. There will be up to two redundant RPs in the system and the RPs will operate in Active-Standby mode for a redundant system.  
- Switch Card (SC): Switch cards sit behind the MPAs and have connections to all MPAs. Switch cards host the NPUs on them. There are up to two switch cards in the system and the SCs work in Active-Standby mode thus delivering data plane redundancy.  
- Power Supplies: Cisco 8608 has 4 power supplies that can provide N+N redundant power to the system. The available options are:  
  o	Pluggable 3.2KW AC   
  o	Pluggable 3.2KW DC   
- Fan Modules: There are 8 Fan Modules in the system with each Fan Module individually removable / replaceable. These provide N+1 fan redundancy to the system. The entire Fan Tray can be removed in case of SC insertion or removal.  

## 8608 Naming Logic     
![naming-1.png]({{site.baseurl}}/images/naming-1.png){: .full}  
![Screenshot 2023-06-28 at 12.36.09 PM.png]({{site.baseurl}}/images/Screenshot 2023-06-28 at 12.36.09 PM.png){: .full}  

![naming-2.png]({{site.baseurl}}/images/naming-2.png){: .full}    
![Screenshot 2023-06-28 at 12.37.14 PM.png]({{site.baseurl}}/images/Screenshot 2023-06-28 at 12.37.14 PM.png){: .full}  



Let us have a quick refresher of the Cisco 8608 PIDs and description.  
  
| Item                 | PIDs            | Description                                                                               |
|----------------------|-----------------|-------------------------------------------------------------------------------------------|
| Chassis              | 8608            | Cisco 8600 - 8 Slot Centralized Chassis                                                   |
| RP                   | 8608-RP         | Cisco 8608 Route Processor                                                                |
| SC                   | 8608-SC0-128    | Cisco 8608 12.8T Switch Card                                                              |
| AC PSU               | 86-3.2KW-AC     | Cisco 8600 3.2KW AC Power Supply Unit                                                     |
| DC PSU               | 86-3.2KW-DC     | Cisco 8600 3.2KW DC Power Supply Unit                                                     |
| Fan Module           | 8608-FAN        | Cisco 8608 Fan Module                                                                     |
| Fan Spinner          | 8608-FS         | Cisco 8608 Fan Spinner (Needed in a chassis with one Switch Card only)                    |
| Redundant Bundle     | 8608-SYS-R      | 2x 8608-RP & 2x 8608-SC0-128 & 8x 8608-FAN & 1x 8608-INSTKT & 1x 8608-RMBRKT              |
| Non-redundant Bundle | 8608-SYS-NR     | 1x 8608-RP & 1x 8608-SC0-128 & 8x 8608-FAN & 1x 8608-FS & 1x 8608-INSTKT & 1x 8608-RMBRKT |
| RP Blank             | 8608-RP-BLANK   | Cisco 8608 RP Blank                                                                       |
| MPA Blank            | 8608-MPA-BLANK  | Cisco 8608 MPA Blank                                                                      |
| Combo MPA            | 86-MPA-14H2FH-M | Cisco 8600 14 x 100G and 2 x 400G Combo Redundant MPA                                     |
| 400G Redundant  MPA  | 86-MPA-4FH-M    | Cisco 8600 4 x 400G Redundant MPA                                                         |
| 10G Redundant MPA    | 86-MPA-24Z-M    | Cisco 8600 24 x 10G/25G/50G Redundant MPA                                                 |  

Table 1. Cisco 8608 PID and description  
{: .text-center}  
 
## Hardware Description  
### Chassis  


![]({{site.baseurl}}/images/figure3.png){: .full}  
Figure 3. Front view of an empty Cisco 8608 chassis  
{: .text-center}  

Cisco 8608 chassis utilizes a state-of-the-art orthogonal design with advanced cooling via front to back airflow, high power capacity and new power supplies. The chassis and all data path components.  

As shown in Figure 3, user needs to insert the followings:  
- Modular Port Adapter(MPA) sits vertically in the front of the chassis
- Route Processor(RP) sits vertically in the front of the chassis
- Switch Card(SC) sits horizontally behind the Fan Trays. Cisco 8608 chassis has two Fan Trays with four Fan Modules on each Fan Tray
- Midplane provides the control path connectivity and power distribution for the chassis



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
- 16 ports of QSFP 100 GbE modules (Default), or
- 2 ports of QSFP-DD 400 GbE modules and 14 ports of QSFP
When port 0 has a 400G QSFP-DD, then the other three ports (2, 3, 4) cannot be used. Similarly, when port 1 has a 400G QSFP-DD, then the other three ports (5, 6, 7) cannot be used.  

**Note**: No restriction in term of power or cooling for 400G ZR/ZR+/Bright ZR+    
{: .notice}  

![figure7.png]({{site.baseurl}}/images/figure7.png){: .full}    
Figure 7. 86-MPA-14H2FH-M  
{: .text-center}  

The second MPA is the 86-MPA-4FH-M. This MPA can be inserted into any slot(slot 0 to slot 7) of the chassis without any restriction. This MPA is a pluggable card that provides 4 interface ports for a maximum total bandwidth of 1.6 Tbps that can support up to:  
- 4 ports of QSFP-DD 400 GbE modules  

**Note**: No restriction in term of power or cooling for 400G ZR/ZR+/Bright ZR+    
{: .notice}  

![figure8.png]({{site.baseurl}}/images/figure8.png){: .full}   
Figure 8. 86-MPA-4FH-M  
{: .text-center}  

And third MPA is 86-MPA-24Z-M. This MPA can be inserted into any slot(slot 0 to slot 7) of the chassis without any restriction. This MPA is a pluggable card that provides 24 interface ports for a maximum total bandwidth of 1.2 Tbps that can support up to:
- 24 ports of SFP 10/25/50 GbE modules


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
- RJ45 console port directly connected to CPU  
- 10/100/1000 Mbps Management Ethernet Port  
- USB 3.0 (Type A)  
- Timing ports: 1588 PTP SFP, SyncE, BITS, GNSS PRTC-A, and GPS(ToD with RJ45 port, 1PPS and 10MHz coax input)  
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

**Note**: The Cisco 8608 chassis doesn't support a mix of AC and DC Power Supply Units (PSUs)  
{: .notice}  

AC Power Module (86-3.2KW-AC)  
- The AC power modules are single feed with 3200W capacity at 220V. They also support operation at 110V with half the capacity.  
- AC power modules are rated at 3200KW maximum at 230VAC high line input, and 1570W maximum at 115VAC low line input.  
- AC nominal range: 100—1230VAC and 2002—40VAC  
- AC full range: 85—132 VAC and 180—264VAC; with extended range to 300VAC  

DC Power Module (86-3.2KW-DC)  
- The DC power modules are dual feed with 3200W capacity at any specified input voltage. DC power modules with single feed at any specified input voltage provide up to 1600W maximum capacity.  
- DC nominal range: -48 to -60VDC  
- DC full range: -40 to -75VDC      

The airflow direction through the Fan Trays and Power Modules on the Cisco 8608 routers are from front to back (port side intake).  

The maximum power available for operations depends on the input power from your power source, the number and output capabilities of power supplies, and the power redundancy mode that you use.   
The following table lists the amount of power available for Cisco 8600 series routers from all available power trays.   

| Total Power Supply | Combined Mode in Watts(No redundancy) | N+1 Redundancy Mode in Watts (with Single Supply Loss) |
|--------------------|---------------------------------------|--------------------------------------------------------|
| 1                  | 3,200                                 | -                                                      |
| 2                  | 6,400                                 | 3,200                                                  |
| 3                  | 9,600                                 | 6,400                                                  |
| 4                  | 12,800                                | 9,600                                                  |  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Cisco8608#sh plat</span>
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
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Cisco8608#sh environment power</span>
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
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Cisco8608#sh environment fan</span>
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
The Cisco 8608 is launched with IOS XR 7.10.1.    

<div class="highlighter-rouge">
<pre class="highlight">
<code>  
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Cisco8608#sh version</span>

Cisco IOS XR Software, <mark>Version 7.10.1.25I LNT</mark>
Copyright (c) 2013-2023 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Thu May 11 23:47:35 UTC 2023
 Build Host   : iox-ucs-058
 Workspace    : /auto/iox-ucs-058-san2/prod/7.10.1.25I.SIT_IMAGE/8000/ws/
 Version      : 7.10.1.25I
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


## Cisco 8608 Slot and Ethernet ports  
Slots and ports numbering are here:  
<b>Slots numbering</b>  
On front view of the Cisco 8608 chassis, left to right direction is MPA0 to MPA7, RP0 to RP1.  

![figure15.png]({{site.baseurl}}/images/figure15.png){: .full}  

Figure 15. Cisco 8608 Slot numbering  
{: .text-center}  

<b>Ports numbering</b>  

![figure16.png]({{site.baseurl}}/images/figure16.png){: .full}  

Figure 16. Cisco 8608 MPA Port numbering  
{: .text-center}  

Cisco 8608 ports numbering convention follows that of physical interfaces as {Type}{R/S/I/P} or {Type}{R/S/I/P/B}  
![RSIPB.png]({{site.baseurl}}/images/RSIPB.png){: .full}  

![Screenshot 2023-06-28 at 12.14.39 PM.png]({{site.baseurl}}/images/Screenshot 2023-06-28 at 12.14.39 PM.png)  

Native Interface Use-case: 86-MPA-4FH-M MPA at port 0 & 1 with native 100 GbE and port 3 & 4 with native 400 GbE on slot 0.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>    
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Cisco8608#sh interfaces brief</span> 

               Intf       Intf        LineP              Encap  MTU        BW
               Name       State       State               Type (byte)    (Kbps)
--------------------------------------------------------------------------------       
         <mark>Hu0/0/0/0</mark>          up          up               ARPA  9000  100000000
         Hu0/0/0/1          up          up               ARPA  9000  100000000
       Hu0/0/0/1.1          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.2          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.3          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.4          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.5          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.6          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.7          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.8          up          up             802.1Q  9004  100000000
       Hu0/0/0/1.9          up          up             802.1Q  9004  100000000
         <mark>FH0/0/0/2</mark>          up          up               ARPA  9000  100000000
         FH0/0/0/3          up          up               ARPA  9000  100000000
</code>
</pre>
</div>  

Breakout Interface Use-case: 86-MPA-14H2FH-M MPA at port 14 with 4x10 GbE on slot 7  

<div class="highlighter-rouge">
<pre class="highlight">
<code>  
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Cisco8608#sh interfaces brief</span> 

               Intf       Intf        LineP              Encap  MTU        BW
               Name       State       State               Type (byte)    (Kbps)
--------------------------------------------------------------------------------       
       <mark>Te0/7/0/14/0</mark>          up          up               ARPA  1514   10000000
       Te0/7/0/14/1          up          up               ARPA  1514   10000000
       Te0/7/0/14/2          up          up               ARPA  1514   10000000
       Te0/7/0/14/3          up          up               ARPA  1514   10000000
</code>
</pre>
</div>  


<b>MPA-IFG mapping</b>  
Cisco 8608 doesn’t have any MPA slot position restriction in the chassis.  
There is one Q200 NPU in each Switch Card.  
 
![]({{site.baseurl}}/images/Screenshot%202023-06-28%20at%2012.16.45%20PM.png){: .full}    
Figure 17. Cisco 8608 MPA- IFG Mapping  
{: .text-center}  

Each SC is connected to MPA 0 to MPA 7. Maximum MPA bandwidth to IFG(Interface Group)s is 1.6 Tbps. And so total 12.8 Tbps duplex per Cisco 8608 system.  

### MPA Details  

<b>86-MPA-14H2FH-M</b>  

This Combo MPA is a Redundant MPA with a front panel has 2x QSFPDDs and 14x QSFP28 ports.  
This MPA is a redundant MPA because it can work with two SC cards with one being active and the other in standby mode.  
This MPA provides a per slot bandwidth of 1.6 Tbps. The QSFPDD ports can support 40/100/400G optics, the QSFP28 ports can support 40/100G optics. 25G and 10G breakout cables are also supported.  

![figure18.jpg]({{site.baseurl}}/images/figure18.jpg){: .full}  

Figure 18. 86-MPA-14H2FH-M Datapath block diagram  
{: .text-center}  

This MPA does not have a CPU and is managed by the RP. This PHY is a single chip, the multi-rate Ethernet PHY that provides the capacity and feature integration required to enable MACsec functionality.  
86-MPA-14H2FH-M can support several interface combinations.  

The first of the combo configurations is 16x100 GbE QSFP, in this configuration the QSFPDD ports are populated with 100G optics not 400G, the QSFP28 ports are populated with 100G optics. This is the default mode(without manual configuration).  
![combo1.jpg]({{site.baseurl}}/images/combo1.jpg){: .full}  

the second of the combo configuration is 2x 400G QSFPDD + 8x 100G QSFP, in this configuration the QSFPDD ports are populated with 400G optics.  

When Port 0 or Port 1 individually changed to 400G mode, disable Port 2/3/4 or Port 5/6/7 automatically.  

The ports that are available in the 2x 400G QSFPDD + 8x 100G QSFP Combo mode are Ports 0/1/8~15 as shown in the figures below. The faceplate will have markings to reflect this.  
![combo2.jpg]({{site.baseurl}}/images/combo2.jpg){: .full}  

The third of the combo configuration is 10/25G breakout solution.  
4x 10G or 4x 25G breakout will not be supported on Port 0 to 9 (PHY 0 & 1). Only on PHY 2’s port 10, 12, and 14 and disabled Port 11, 13, and 15.  
![combo3.jpg]({{site.baseurl}}/images/combo3.jpg){: .full}  


<b>86-MPA-4FH-M</b>  
This MPA is a Redundant MPA with a front panel consisting of 4x QSFPDD ports.  This is a redundant MPA because it can work with to two SCs with one being active and the other in standby mode.  
This MPA provides a per slot bandwidth of 1.6Tbps. The QSFPDD ports can support 40/100/400G optics and support fully Breakout solutions without any port restriction.  

![figure19.jpg]({{site.baseurl}}/images/figure19.jpg){: .full}  

Figure 19. 86-MPA-4FH-M Datapath block diagram  
{: .text-center}  


<b>86-MPA-24Z-M</b>  
This MPA is a Redundant MPA with a front panel consisting of 24x SFP56 ports.  This MPA is a redundant MPA because it can work with to two SC with one being active and the other in standby mode.  
This MPA provides a per slot bandwidth of 1.2 Tbps. The QSFP56 ports can support 10/25/50G optics and can’t support any Breakout solution.  


![figure20.jpg]({{site.baseurl}}/images/figure20.jpg){: .full}  

Figure 20. 86-MPA-24Z-M Datapath block diagram  
{: .text-center}  

To identify the optic types supported on the Cisco 8608, please check [[the TMG matrix](https://tmgmatrix.cisco.com/)]      
It contains details on the connector types, the reach, the minimum release required, etc.  

<b>Maximum ports density supported on Cisco 8608</b>  

|          | 10 GbE | 25 GbE | 40 GbE | 50 GbE | 100 GbE | 400 GbE |
|----------|--------|--------|--------|--------|---------|---------|
| Native   | 192    | 192    | 128    | 192    | 128     | 32      |
| Breakout | 128    | 128    | N/A    | N/S    | 128     | N/A     |  
  
N/A: Not applicable – N/S: Not supported  

Math is the following:  
- 10 GbE  
  - Use with 86-MPA-24Z-M as native with SFP+, fully populated with this MPA into 8 MPAs slots. 8x MPAs and 24x SFP+ on each MPA: 192x 10 GbE total    
  - Use with 86-MPA-4FH-M as breakout, fully populated with this MPA into 8 MPAs slots. 4x (4x 10 GbE) x 8 MPAs: 128x 10 GbE total  
- 25 GbE  
  - Use with 86-MPA-24Z-M as native with SFP28, fully populated with this MPA into 8 MPAs slots. 8 MPA x 24x SFP28 on each: 192x 25 GbE total  
  - Use with 86-MPA-4FH-M as breakout, fully populated with this MPA into 8 MPAs slots. 4x (4x 25 GbE) x 8 MPAs: 128x 25 GbE total  
- 40 GbE  
  - Use with 86-MPA-14H2FH-M as native with QSFP+, fully populated with this MPA into 8 MPAs slots. 8 MPAs x 16x QSFP+ on each: 128x 40 GbE total  
- 50 GbE  
  - Use with 86-MPA-24Z-M as native with SFP56, fully populated with this MPA into 8 MPAs slots. 8 MPAs x 24x SFP56 on each: 192x 50 GbE total  
- 100 GbE  
  - Use with 86-MPA-14H2FH-M as native with QSFP28, fully populated with this MPA into 8 MPAs slots. 8 MPA x 16x QSFP+ on each: 128x 100 GbE total  
  - Use with 86-MPA-4FH-M as breakout, fully populated with this MPA into 8 MPAs slots. 8 MPAs x 4x (4x 100 GbE) on each: 128x 100 GbE total  
- 400 GbE  
  - Use with 86-MPA-4FH-M as native with QSFPDD, fully populated with this MPA into 8 MPAs slots. 8 MPAs x 4x QSFPDD on each: 32x 100 GbE total  

### Switch Card (SC) Details   
Switch Card has a 12.8T bandwidth using the Cisco Silicon One Q200 NPU at an operating data rate of 56 Gbps per lane. It supports 8 pluggable modular port adapter (MPA) cards each supporting up to 1.6 Tbps. There can be two SCs in a system that provides full data plane redundancy in an active-standby mode.  

![8608 block.jpg]({{site.baseurl}}/images/8608 block.jpg){: .full}  
Figure 21. Cisco 8608 block diagram  
{: .text-center}  

Cisco Silicon One Q200’s brief features are following:    
- 12.8 Tbps full-duplex and 8.1 Bpps Forwarding Capacity NPU  
- 256 56G SerDes with each capable of operating at 10G/25G/50G using NRZ or PAM4 modulation   
- Flexible port configuration supporting 10/25/40/50/100/200/400 Gbps   
- 108 MB fully shared on-die packet buffer   
- Expandable in-package packet buffer using 8 GB HBM  
- On-chip, high-performance, P4-programmable host NPU for high-bandwidth offline packet processing  
- Multiple embedded processors for CPU offloading  
- 56K VoQ(Virtual Output Queue)   
- Dedicated 108 Counters banks (860K counters)   

For more understanding on Cisco Silicon One Q200 refer to [Cisco Silicon One Q200-Info](https://www.cisco.com/c/en/us/solutions/collateral/silicon-one/datasheet-c78-744312.html).  


### Redundancy Details  
Within the system even though RP0, RP1, SC0, SC1 are 4 different cards, from redundancy standpoint system software would consider RP0-SC0 together as Active Pair and RP1-SC1 as Backup Pair.  

There’s supported and unsupported configuration in Cisco 8608.  

| System Type   | Supported configuration | Unsupported configuration |
|---------------|-------------------------|---------------------------|
| Redundant     | RP0-SC0, RP1-SC1        | RP0-SC1                   |
|               |                         | RP1-SC0                   |
|               |                         | RP0-SC0, RP1 without SC1  |
|               |                         | RP1-SC1, RP0 without SC0  |
|---------------|-------------------------|---------------------------|
| Non-Redundant | RP0-SC0                 | RP1-SC1                   |
|               | FS1 without RP1         | RP1-SC0                   |
|               |                         | RP0-SC1                   |  


For Non-Redundant 8608 configuration, user can use “<b>8608-SYS-NR</b>” PID in the order tool.
 8608 system is shipped with only one RP, one SC, one FS.    
For Redundant 8608, user can use “<b>8608-SYS-R</b>” PID in the order tool. System is shipped with two RPs, two SCs.  

Let’s start with the default state when system boot up under redundant system.  
 RP0-SC0 together as one Domain 0(Active Pair) and RP1-SC1 as the other Domain 1(Standby Pair).  
 
![redundancy-1.png]({{site.baseurl}}/images/redundancy-1.png){: .full}  
Figure 22. Default Redundant state in Cisco 8608  
{: .text-center}  

In this default scenario, RP0 gains mastership and RP1 is in Backup state.
Both SC0 and SC1 are packet processing and forwarding capability.  

If the RP0 fatal faults or removal,  

![RP0-Fail.jpg]({{site.baseurl}}/images/RP0-Fail.jpg){: .full}  
Figure 23. RP0 failure scenario in Cisco 8608  
{: .text-center}  

Standby RP immediately gains mastership via HW arbitration mechanism. MPAs switchover data path to RP1-SC1.  

![SC0_Fail.jpg]({{site.baseurl}}/images/SC0_Fail.jpg){: .full}  
Figure 24. SC0 failure scenario in Cisco 8608  
{: .text-center}  

Another scenario is the failure of the SC0 within Active pair, Active RP’s shelfmgr relinquish mastership if standby RP present and ready. Trigger SC reload regardless.  

No traffic drop is expected during RP/SC Failover.  

### Packet flow in the redundancy system  

![figure21.jpg]({{site.baseurl}}/images/figure21.jpg){: .full}  
Figure 25. Data path flow with Redundant System  
{: .text-center}  

1.	Ingress traffic coming from the network into the MPA  
2.	PHY is bi-casted towards Q200s on both Active and Standby Pairs  
3.	Q200s on both active and standby process the received packets  
4.	Both Q200s forward packets towards the MPA that has the destination port    
5.	In the egress path, PHY will forward the packets from active SC0  
6.	PHY will drop packets coming from standby SC1  

![figure22.jpg]({{site.baseurl}}/images/figure22.jpg){: .full}  
Figure 26. Data path flow after Active Pair failover  
{: .text-center}  

## Conclusion  
This article covered Cisco 8608 deep-dive level architecture.  

## Reference   
- Datasheet: https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/datasheet-c78-742571.html 
- Hardware Installation Guide: Need to add  
- Power calculator: Need to add  
- Cisco 8000 Configuration guide: https://www.cisco.com/c/en/us/support/routers/8000-series-routers/products-installation-and-configuration-guides-list.html   
- Cisco Optics-to-Device Compatibility Matrix tool: https://tmgmatrix.cisco.com/  

If you want to reach out for comments, feedback or questions, send us a mail at:
[ask-sp-routing@cisco.com](mailto:ask-sp-routing@cisco.com) and [8000-tme@cisco.com](mailto:8000-tme@cisco.com)
