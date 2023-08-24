---
published: false
date: '2023-08-22 12:04 +0530'
title: 'Cisco 8000 Q100/Q200 Hardware resources: Encapsulation DB for label transport'
author: 'Ram Mohan A. M, Deepak Balasubramaniam'
excerpt: >-
  This writeup explains label resource allocation details for labelled transport
  application for Cisco 8000 routers
tags:
  - iosxr
  - cisco
  - Cisco 8000
  - LDP
---

## Introduction  

With Cisco 8000 routers gaining popularity across various service provider and cloud customers, it’s very important to understand how the system behaves when running labelled transport and services. The reason being the labelled applications are hardware resource intensive in the systems built with single stage forwarding architecture. 

In this article we will focus on the basic label forwarding transport LDP, SR-MPLS (with/without ECMP, Bundles) and look into details on the hardware resource usage. Main focus will be on the Encapsulation Database.  Though there are other hardware databases used by LDP/SR, Encap DB is an important one for all labelled applications. 

Also, in this article we will focus on the Silicon One Q100 and Q200 based systems only. 

Resource utilization for the next generation of Silicon One Asics is different and we will have another XR doc article coming up on that. 


## Cisco 8000 PIDs with Q100/Q200 

 This table gives a table view of current Cisco 8000 products mapping to Q100/Q200 
 
 
 
 
 
 

## Hardware resources at high level
Fundamentally silicon One ASIC is built with 6 slices where each slice is a set of parallelly processing engines for different packet processing functionalities like: IFG- provisions MAC-orts, RxPP – receive packet processor, TxPP- Transmit packet processor and all traffic managing entities (VOQs, schedulers etc..). And number of slices can differ between different ASIC variants keeping fundamental architecture intact. 

 

Primarily there are 3 types of hardware memories at high level which are below, 

  - TCAM 
  - SRAM 
  - HBM 

And these memories area globally shared across all slices. And there are multiple data bases associated with these memories which get accessed at different stages of packet processing.  

 

Major data base classification is as below, 
- Prefixes storage


LPM 

Prefixes, MACs, local labels etc. 

CEM 

Remote labels for different services labels 

Egress EMs (Large encap & small encap) 

Next hop groups and ECMP members 

Stage1lb: group & members 

Stage2lb: group & members 

 

 

This write up is about egress encapsulation management, and monitoring. 

 

Label encap resource consumption can be per slice level , per slice-pair or per device/NPU level based on different label applications implemented in the system. Below table brief on the resources associated with label programming and its scope of programming, 

![Screenshot 2023-08-22 at 12.15.37 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.15.37 PM.png)


Lets understand Cisco 8000 slice-pair concept before get into the details of egress large encap resource details.

### Slices & Slice-pair concepts

For a Fixed or Centralised systems all 6 slices on the NPU is available for network facing interfaces but same time for a Distributed system only 3 slices are available for network facing interfaces and other 3 slices are used for fabric facing interfaces. 

Below pictures depict the slices for fixed system and distributed system (here representing 88-LC0-34H14FH line card which has 2 NPUs),

![Screenshot 2023-08-22 at 12.19.29 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.19.29 PM.png)

![Screenshot 2023-08-22 at 12.26.02 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.26.02 PM.png)



Fixed systems will have 3 slice-pairs (6 n/w slices) & Distributed systems will have 2 slice-pairs (3 n/w slices)

  - If interface belongs to slice # 0,1 then its slice-pair 0
  - If interface belongs to slice # 2,3 then its slice-pair 1
  - If interface belongs to slice # 4,5 then its slice-pair 2


How to find the interface mapping to NPU/Slice/IFG? See below example which is from a distributed system,


![Screenshot 2023-08-22 at 12.27.52 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.27.52 PM.png)

## Understand Encapsulation databases & Label types

There are 2 types of encapsulation database on Q100/Q200 ASIC based Cisco 8000 systems,

  - Large Encap DB: holds MPLS out label which is the remote label received from remote peer nodes
  - Small Encap DB: holds MPLS remote labels which is recursively resolved for overlay service labels.
	
There are 2 types of labels,
-	**Remote label**: label received from remote peer
      - It can be transport labels like LDP, SR, RSVP-TE etc..
      - It can be service labels like L3VPN , L2VPN, BGP-LU etc..
-	**Local label**: label assigned by the system locally 

Once system receives remote label from peer-device a corresponding local label get assigned for the same forwarding entity. And system advertises the local label further to its peer-devices.

Remote labels are managed in egress encapsulation (EM) databases and local labels are managed in centra exact match (CEM) data bases. Whereas local labels which are looked up at the ingress packet termination stage are programmed in CEM table which is a global table for whole NPU system.

### Egress Large EM on Q100 & Q200 systems

**Q100 based systems**: The size of large encap table is 32K/slice-pair 
  -	On *Fixed systems*, this table is divided into three parts, one for each slice-pair. So, each slice-pair has encap table of size 32K(Q100). And total per NPU is 96k
  -	However, on *distributed systems*,  the NPUs in line cards will have only 3 n/w slices (1.5 slice-pairs) so effectively 64K encap entries exist per NPU.

**Q200 based systems**: Large encap table size is 64K/slice pair
  - On *fixed system* , this table is divided into three parts, one for each slice-pair. So, each slice-pair has encap table of size 64K . And total per NPU is 192K per NPU
  - On *distributed systems*, this table size can go up to 128K per NPU (1.5 slice-pairs).
  
**Following illustrations explains how Egress Large encap programming varies for different topologies:**




![Screenshot 2023-08-22 at 12.48.15 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.48.15 PM.png)


-	In Topo-1, the egress interface facing the MPLS cloud is hosted on slice2 (slice-pair1)
-	System has received 10k labelled prefixes.
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pair, which is hosting the egress interface, here slice-pair1
-	Local labels associated with each remote labels get programmed in CEM.




![Screenshot 2023-08-22 at 12.49.28 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.49.28 PM.png)

-	In Topo-2, System has received 10k labelled prefixes over 2 ECMP links where both links are hosted on same slice-pair (slice-pair1)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pair which is hosting the egress interfaces. So 20k encap entries get programmed on slice-pair1 since there are 2 ECMPs links on that slice-pair.
-	Local labels associated with each remote labels get programmed in CEM.



![Screenshot 2023-08-22 at 12.50.17 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.50.17 PM.png)

-	In Topo-3, System has received 10k labelled prefixes over 2 ECMP links which are spread across 2 different slice-pairs (slice-pair0 & slice-pair2)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pairs which are hosting the egress interfaces. Here there is 1 link each on slice-pair0 & 2. So 10K encap entries get programmed on slice-pair0 & slice-pair2
-	Local labels associated with each remote labels get programmed in CEM


![Screenshot 2023-08-22 at 12.50.47 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.50.47 PM.png)

-	In Topo-4, System has received 10k labelled prefixes over a bundle interface which has 2 member links distributed across 2 slice-pairs (slice-pair0 & 1)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pairs which is hosting the bundle member links. So 10k encap entries get programmed on each slice-pair0 & 1 . 
-	Local labels associated with each remote labels get programmed in CEM


![Screenshot 2023-08-22 at 12.51.23 PM.png]({{site.baseurl}}/images/Screenshot 2023-08-22 at 12.51.23 PM.png)


-	In Topo-5, System has received 10k labelled prefixes over a bundle interface which has 2 member links hosted on same slice-pair (slice-pair1)
-	10K prefixes get programmed in LPM database.
-	Remote labels associated with the prefixes get programmed in egress-large-encap database on the slice-pair which is hosting the bundle member links. So only 10k encap entries get programmed on slice-pair1. Member links from same slice-pair consume single encap entry for each labelled prefix.
-	Local labels associated with each remote labels get programmed in CEM







