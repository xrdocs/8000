---
published: false
date: '2023-08-23 10:13 +0200'
title: >-
  Solving MPLS Traffic Engineering Bin-Packing problem with Cisco TE++
  Auto-Capacity
tags:
  - iosxr
  - Cisco 80000
  - TE++
  - Auto-bandwidth Capacity
position: hidden
author: Frederic Cuiller
excerpt: >-
  This blog details Cisco’s TE++ Auto-bandwidth Capacity feature using a Cisco
  8000 environment.
---
# Introduction

Building point-to-point Traffic Engineering LSPs can lead to inefficient total network bandwidth utilization. Cisco’s TE++ Auto-bandwidth Capacity allows an ingress router to interact with the other routers in the network, locate additional available bandwidth and automatically create parallel LSPs to acquire all the necessary network resources possible until they are no longer needed.  This blog details one of the possible applications of this feature using a Cisco 8000 environment.

This blog details Cisco’s TE++ Auto-bandwidth Capacity feature using a Cisco 8000 environment.

# Problem Description

In MPLS Traffic Engineering, a Bin-Packing problem is the inability to efficiently allocate the total reserved bandwidth. Observe the following:

![mpls-te-topology.png]({{site.baseurl}}/images/mpls-te-topology.png)

Router A sends a 300Mbps RSVP reservation into the network destined somewhere beyond R4. Router R1 has three paths to pick from:
1. Path-1: R1-R4, with 1Gbps of reservable bandwidth
2. Path-2: R1-R2-R4, with 1.5Gbps of reservable bandwidth
3. Path-3: R1-R3-R4, with 1.5Gbps of reservable bandwidth

Let’s say the network is setup that Path-1 is chosen.  700Mbps of reservable bandwidth remains on Path-1. Now Router B sends an 800Mbps RSVP reservation into the network.  Since Path-1 is out of reservable capacity (300Mbps + 800Mbps > 1Gbps), Path-2 (could be Path-3, but it does not matter) is chosen and 700Mbps remain.  When an additional 800Mbps flow is requested, we face a similar problem on Path-2 (800Mbps + 800Mbps > 1.5Gbps), Path-3 will be the only remaining choice.  Given our demands we have booked 1.9Gbps of reservations on a 3.5Gbps of available capacity.  

Should another 800Mbps be requested on the network, we will not be able to turn up this LSP on any of these links, even though we have sufficient resources to do so. You can do your own math on your network, but we are now sitting on a 46% of wasted unused reservation bandwidth in our scenario.  We cannot efficiently **“pack the bin.”**  We would like to change this and get as close as possible to make dynamic provisions for the entire 3.5Gbps of available reservations that will scale up and down without operator intervention.

# Cisco's TE++ Auto-Capacity Demonstration

We do this with a feature called “auto-capacity”.  You will find references to Container LSP or TE++ in other vendors’ documents, but the concept is the same. As traffic demands grow and shrink, we will use LSP splitting functionality to efficiently fill our total reservable bandwidth, i.e bin-packing the links.  

We run this customer demo on Cisco 8000 series routers which are powered by Silicon One. The topology is the following:



Here is the relevant part of R1’s configuration where all the magic (dynamic tunnels) happens:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
mpls traffic-eng
 interface FortyGigE0/3/0/0
 !
 interface FortyGigE0/3/0/3
 !
 interface HundredGigE0/3/0/6
 !        
 reoptimize 60
 auto-bw collect frequency 1
 named-tunnels
  tunnel-te tesla
   path-option 10
    preference 200
    computation dynamic
   !
   auto-bw ---------------------------------- Adjust bandwidth needs dynamically
    overflow threshold 30 min 10 limit 1
    adjustment-threshold 20 min 10
    application 5 --------------------------- Evaluate every 5min
    auto-capacity --------------------------- Adjust capacity
     merge-bandwidth 150000 ----------------- Combine LSPs at 150Mbps
     split-bandwidth 300000 ----------------- Split LSPs at 300Mbps
     max-clones 15 -------------------------- Create 15 child tunnels maximum
     min-clones 0 --------------------------- Start with 0 tunnel
     nominal-bandwidth 200000 --------------- Base bandwidth is 200Mbps
    !
   !
   signalled-bandwidth 200000
   autoroute announce ----------------------- Get TE-tunnel into IGP
   !      
   destination 10.100.100.4 ----------------- R4 is tail-end of the tunnel
  !
 !
!
</code>
</pre>
</div>

Please note that you will have to use different configuration values to suit your network needs – the ones we provide here can help you see the full cycle of tunnel adjustments in less than 1 hour of your time as the network adjusts to various traffic demands.

What are we doing here?  We replaced routers A and B with a traffic generator.  The logic is the following:

1. Announce RSVP tunnel Cisco into IGP calculations
2. Use 200Mbps as nominal bandwidth
3. When the traffic grows to 300Mbps, start splitting LSPs
4. Create up to 15 child tunnels (16 total if you add the parent) as traffic demands grow
5. As traffic demands diminish, start merging tunnels together
6. Evaluate this every 5 minutes

Let us see this in action: 
Sending ~200Mbps (on the interface and into tunnel Cisco) into the network:



