---
published: false
date: '2023-08-22 12:04 +0530'
title: >-
  Cisco 8000 Hardware resources deep dive Part-1: Encapsulation database for
  label transport
author: 'Ram Mohan A. M, Deepak Balasubramaniam'
excerpt: >-
  This writeup explains label resource allocation details for labelled transport
  application for Cisco 8000 routers
tags:
  - iosxr
  - cisco
  - Cisco 8000
  - LDP
position: hidden
---

{% include toc icon="table" title="Table of Contents" %}

|Ram Mohan, Technical Leader, Technical Marketing (raam@cisco.com)|  
|Deepak Balasubramanian, Technical Leader, Technical Marketing (deebalas@cisco.com)|  

## Introduction  

With Cisco 8000 routers gaining popularity across various service providers and cloud customers, it’s very important to understand how the system behaves when running labelled transport and services. The reason being the labelled applications are hardware resource intensive, especially with the systems built with single stage forwarding architecture.

In this article we will focus on the basic label forwarding transport with & without ECMP along with bundles and look into details on the hardware resource usage. Main focus will be on the Encapsulation Databases.

Also, in this article we will focus on the Silicon One **Q100 and Q200 based 8200/8600/8800** systems only.
Resource utilization for the next generation of Silicon One Asics (P100, K100) is different and we will have another XR docs article focusing on that.



## Cisco 8000 PIDs with Q100/Q200 

 This table gives the view of current Cisco 8000 products mapping to Q100/Q200 
 
 
|**Forwarding Asics** | **Cisco 8000 PIDs**  
|------------------------|------------------------------------------------------------------|
| Q100          | **Fixed**: 8201, 8202 ,  															**8800-Line cards:** 8800-LC-36FH, 8800-LC-48H
| Q200      | **Fixed:** 8201-24H8FH, 8202-32FH(-M), 8608  , 										**8800-Line cards:** 88-LC0-36FH(-M), 88-LC0-36H14FH


When we compare the forwarding Asics Q100 & Q200, they are very similar in architecture in terms of hardware components and packet forwarding. But there are differences in terms of the number of 50G serdes links which brings changes in forwarding throughput between the Asics. 
 
|   **Q100** | **Q200**  
|------------------------|------------------------------------------------------------------|
| •	10.8 Tbps                             | •	12.8 Tbps 
| •	6 Slices with 216 x 50G serdes  | •	6 Slices with 256 x 50G serdes
| •	2 x IFG per slice with 18 x 50G serdes per IFG            | •	2 x IFG per slice with 16 x 50G (4 IFGs) or 24 x 50G (8 IFGs)
| •	36MB on-die packet buffer (SMS)            | •	108MB on-die packet buffer (SMS)
| •	HBM            | •	HBM*

_*No HBM on Q200L based systems._


Also, there are differences in the hardware resources which will bring in scale differences between the two Asics.

 
![Q100vsQ200_resource-writeup.png]({{site.baseurl}}/images/Q100vsQ200_resource-writeup.png){: .align-center}

## Hardware resources at high level
Fundamentally, silicon One ASIC is built with 6 slices where each slice has a set of parallel processing engines for different packet processing functionalities like: IFG - provisions MAC-orts, RxPP – receive packet processor, TxPP- Transmit packet processor and Traffic Manager - VOQs, schedulers. 

And number of slices can differ between different ASIC variants keeping fundamental architecture intact.

<<6 slice pic>>

Primarily there are 3 types of hardware memories at a high level, 

  - TCAM 
  - SRAM 
  - HBM 

Some of these memories area globally shared across all slices while some are per slice level or slice-pair level. And there are multiple data bases associated with these memories which get accessed at different stages of packet processing.  

 

Major data base classification is as below,

|**Databases** | **Application**  
|------------------------|------------------------------------------------------------------|
| LPM          | Prefixes storage
| HBM | LPM FIB table expansion
| CEM      | Prefixes, MACs, local labels etc
| Egress EMs (Large encap & small encap)       | Remote labels for different services labels 
| Stage1lb/Stage2lb: group & members | Next hop groups and ECMP members
| TCAM | Packet classificaion (ACL,QOS,LPTS)

 
## Single Stage packet Forwarding: Databases view



### What is Single stage forwarding?

Forwarding lookup and all the forwarding decisions are done in the ingress of the packet processor (RxPP) and the packets are properly queued to the VOQ of the corresponding egress interface.
In the RxPP during forwarding lookup and resolution, there will be system headers added which have pointers for the encapsulation action on the egress of packet processor (TxPP).
Once the packet arrives at the TxPP stage, pointers in the system header will be used to fetch the right encapsulation information (labels, MAC) from the EM database which is used to build the network headers before sending the packets out of the egress interface.
 


![Forwarding-pipeline-reources-DB.png]({{site.baseurl}}/images/Forwarding-pipeline-reources-DB.png) {: .align-center}


To simplify the flow
- Packet from network interface (NIF) is received by the RX NPE in the ingress.
- Forwarding lookup is done on LPM (IP lookup) or CEM (ex, for labels) and we derive the next hop (NH) and ECMP info.
- LB ECMP group resolution helps to arrive at TM VoQ for the destination port along with egress encap pointers.
- Packets with system headers (carrying encap pointers) are enqueued into the VoQs
- Packets are received on the Tx NPE, system headers are referenced to get the encap pointers in the Encapsulation database to derive the label and the MAC information.
- Packets are reconstructed with proper network headers and send out of the corresponding egress interface.


## Label encap resource usage

large & small, resource as seen below


some appln programs at slice-pai and some on all slice-pairs


Label encap resource consumption can be per slice level , per slice-pair or per device/NPU level based on different label applications implemented in the system. Below table brief on the resources associated with label programming and its scope of programming, 


![resource-table-1.png]({{site.baseurl}}/images/resource-table-1.png){: .align-center}


**This write up is about _egress large encapsulation_ resource usage and monitoring**

 
Lets understand Cisco 8000 slice-pair concept before get into the details of egress large encap resource details.



### Slices & Slice-pair concepts

For a Fixed or Centralised systems all 6 slices on the NPU is available for network facing interfaces but same time for a Distributed system only 3 slices are available for network facing interfaces and other 3 slices are used for fabric facing interfaces. 

Below pictures depict the slices for fixed system and distributed system (here representing 88-LC0-34H14FH line card which has 2 NPUs),


![resource-slicepair-1.png]({{site.baseurl}}/images/resource-slicepair-1.png){: .align-center}

**Illustration**

![resource-slicepair-2.png]({{site.baseurl}}/images/resource-slicepair-2.png){: .align-center}


Fixed systems will have 3 slice-pairs (6 n/w slices) & Distributed systems will have 2 slice-pairs (3 n/w slices)

for fixed systems,
  - If interface belongs to slice # 0,1 then its slice-pair 0
  - If interface belongs to slice # 2,3 then its slice-pair 1
  - If interface belongs to slice # 4,5 then its slice-pair 2

for distributed systems,
  - If interface belongs to slice # 0,1 then its slice-pair 0
  - If interface belongs to slice # 2 then its slice-pair 1
  
How to find the interface mapping to NPU/Slice/IFG ? 


![resource-voq-output.png]({{site.baseurl}}/images/resource-voq-output.png){: .align-center}

Above example is from a distributed system.

## Understanding Encapsulation databases & Label types

There are 2 types of encapsulation database on Q100/Q200 ASIC based Cisco 8000 systems,

  - **Large Encap DB**: Holds MPLS out labels (transport & services) which is the remote label received from remote peer nodes (LDP/SR/RSVP-TE, L2VPN/L3VPN)
  - **Small Encap DB**: Holds MPLS remote labels for **underlay transport** with recursive overlay transport (BGP-LU over **LDP**/**RSVP-TE**, LDP over **RSVP-TE**)
	
There are 2 types of labels,
-	**Remote label**: label received from remote peer
      - It can be transport labels like LDP, SR, RSVP-TE etc.
      - It can be service labels like L3VPN , L2VPN, BGP-LU etc.
-	**Local label**: label assigned by the system locally 

Remote labels are managed in egress encapsulation (EM) databases and local labels are managed in centra exact match (CEM) data bases. 
Local labels are looked up from CEM in the ingress termination stage and remote labels are derived from egress encapsulation stage from EM database.

### Egress Large EM on Q100 & Q200 systems

**Q100 based systems**: The size of large encap table is 32K/slice-pair 
  -	On *Fixed systems*, this table is divided into three parts, one for each slice-pair. So, each slice-pair has encap table of size 32K(Q100). And total per NPU is 96k
  -	However, on *distributed systems*,  the NPUs in line cards will have only 3 n/w slices (1.5 slice-pairs) so effectively 64K encap entries exist per NPU.

**Q200 based systems**: Large encap table size is 64K/slice pair
  - On *fixed system* , this table is divided into three parts, one for each slice-pair. So, each slice-pair has encap table of size 64K . And total per NPU is 192K per NPU
  - On *distributed systems*, this table size can go up to 128K per NPU (1.5 slice-pairs).
  
**Following illustrations explains how Egress Large encap programming varies for different topologies:**


## MPLS transport label Scenario without ECMP


![resource-topo-1.png]({{site.baseurl}}/images/resource-topo-1.png){: .align-center}


-	In Topo-1, the egress interface facing the MPLS cloud is hosted on slice2 (slice-pair1)
-	System has received 10k labelled prefixes.
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pair, which is hosting the egress interface, here slice-pair1
-	Local labels associated with each remote labels get programmed in CEM.

## MPLS transport label Scenario with ECMP on same Slice-pair

![resource-topo-2.png]({{site.baseurl}}/images/resource-topo-2.png){: .align-center}


-	In Topo-2, System has received 10k labelled prefixes over 2 ECMP links where both links are hosted on same slice-pair (slice-pair1)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pair which is hosting the egress interfaces. So 20k encap entries get programmed on slice-pair1 since there are 2 ECMPs links on that slice-pair.
-	Local labels associated with each remote labels get programmed in CEM.

## MPLS transport label Scenario with ECMP across different Slice-pairs

![resource-topo-3.png]({{site.baseurl}}/images/resource-topo-3.png){: .align-center}


-	In Topo-3, System has received 10k labelled prefixes over 2 ECMP links which are spread across 2 different slice-pairs (slice-pair0 & slice-pair2)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pairs which are hosting the egress interfaces. Here there is 1 link each on slice-pair0 & 2. So 10K encap entries get programmed on slice-pair0 & slice-pair2
-	Local labels associated with each remote labels get programmed in CEM

## MPLS transport label Scenario with Bundle members across different Slice-pairs

![resource-topo-4.png]({{site.baseurl}}/images/resource-topo-4.png){: .align-center}


-	In Topo-4, System has received 10k labelled prefixes over a bundle interface which has 2 member links distributed across 2 slice-pairs (slice-pair0 & 1)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pairs which is hosting the bundle member links. So 10k encap entries get programmed on each slice-pair0 & 1 . 
-	Local labels associated with each remote labels get programmed in CEM

## MPLS transport label Scenario with Bundle members on same Slice-pair

![resource-topo-5.png]({{site.baseurl}}/images/resource-topo-5.png){: .align-center}


-	In Topo-5, System has received 10k labelled prefixes over a bundle interface which has 2 member links hosted on same slice-pair (slice-pair1)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pair which is hosting the bundle member links. So only 10k encap entries get programmed on slice-pair1. Member links from same slice-pair consume single encap entry for each labelled prefix.
-	Local labels associated with each remote labels get programmed in CEM

## Deployment Recommendations (with label transport Q100/Q200)
-Spread the ECMP links accross different slice-pairs
-Keep the bundle members localized to the same slice pair as much as possible
This helps with optimized usage of Encapsualtion database for labelled transport.


## Monitoring labels and encap resources


![resource-3-rtr-topo.png]({{site.baseurl}}/images/resource-3-rtr-topo.png){: .align-center}

In above topology, 
-	R1, R2 & R3 form OSPF+LDP peering OR OSPF+SR
-	R3 advertise 10k labelled prefixes to R2 and R2 eventually to R1
-	R1:R2 link is terminating on slice-pair2 on R1

## Resource check: Before Label advertisements
Lets check the egress encap data base on the UUT (R1) in above topology before R1 received labelled prefixes from R2,

![resource-emlarge-b4-programing.png]({{site.baseurl}}/images/resource-emlarge-b4-programing.png){: .align-center}

What is seen in the resource table above,
-	OOR summary shows the OOR state and threshold levels. 
-	Egress encap entries are programmed as ‘lspnh’ entries in OFA table. Its zero as this o/p is before R1 programmed the entries
-	Hardware usage shows the actual HW programmed entries at slice-pair level.
-	There are some default entries get programmed in HW as seen above. 

## Resource check: After Label advertisements

 Let’s advertise 10k labelled prefixes from R3 to R2 over LDP, eventually to R1.

MPLS forwarding database on R2 is seen as below. Outgoing labels are seen as ‘unlabeled’ because R3 is directly connected neighbor for those prefixes and R2 is PHP for those prefixes. And as seen in CEF o/p there are 10k prefixes updated on R2



![cef-mpls.png]({{site.baseurl}}/images/cef-mpls.png){: .align-center}


Let’s analyse Egress Large Encap resource utilization on R1-UUT , 

![resource-aftr-emlarge-programing.png]({{site.baseurl}}/images/resource-aftr-emlarge-programing.png){: .align-center}

-	10k encap entries are programmed in slice-pair2 (egress interface which is pointing the nexthop is hosted on slice-pair2) in hardware.
-	These 10k entries are for outgoing LDP labels. Hence the resource consumption scope is at slice-pair level.

Same label allocation model applies to SR-MPLS transport scenarios as well.

## Conclusion
With this we come to the end of this article where we dicussed in details on the harware resource usage on Q100/Q200 for the labelled transport with specific focus on Egress large encapsulation database.
In the following articles we will deepdive into other hardware resources and different applications.
