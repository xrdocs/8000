---
published: true
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
author: Bradley Riapolov
excerpt: >-
  This blog details Cisco’s TE++ Auto-bandwidth Capacity feature using a Cisco
  8000 environment.
---
{% include toc title="MPLS-TE Auto-Bandwidth TE++ on Cisco 8000" %}

# Introduction

Building point-to-point Traffic Engineering LSPs can lead to inefficient total network bandwidth utilization. Cisco’s TE++ Auto-bandwidth Capacity allows an ingress router to interact with the other routers in the network, locate additional available bandwidth and automatically create parallel LSPs to acquire all the necessary network resources possible until they are no longer needed.  This blog details one of the possible applications of this feature using a Cisco 8000 environment.

This blog post details Cisco’s TE++ Auto-bandwidth Capacity feature using a Cisco 8000 environment.

# Problem Description

In MPLS Traffic Engineering, a Bin-Packing problem is the inability to efficiently allocate the total reserved bandwidth. Observe the following:

![mpls-te-topology.png]({{site.baseurl}}/images/mpls-te-topology.png)

Router A sends a 300Mbps RSVP reservation into the network destined somewhere beyond R4. Router R1 has three paths to pick from:
1. Path-1: R1-R4, with 1Gbps of reservable bandwidth
2. Path-2: R1-R2-R4, with 1.5Gbps of reservable bandwidth
3. Path-3: R1-R3-R4, with 1.5Gbps of reservable bandwidth

Let’s say the network is setup that Path-1 is chosen.  700Mbps of reservable bandwidth remains on Path-1. Now Router B sends an 800Mbps RSVP reservation into the network.  Since Path-1 is out of reservable capacity (300Mbps + 800Mbps > 1Gbps), Path-2 (could be Path-3, but it does not matter) is chosen and 700Mbps remain.  When an additional 800Mbps flow is requested, we face a similar problem on Path-2 (800Mbps + 800Mbps > 1.5Gbps), Path-3 will be the only remaining choice.  Given our demands we have booked a 1.9Gbps of reservations on a 3.5Gbps of available capacity.  

Should another 800Mbps be requested on the network, we will not be able to turn up this LSP on any of these links, even though we have sufficient resources to do so. You can do your own math on your network, but we are now sitting on a 46% of wasted unused reservation bandwidth in our scenario.  We cannot efficiently **“pack the bin.”**  We would like to change this and get as close as possible to make dynamic provisions for the entire 3.5Gbps of available reservations that will scale up and down without operator intervention.

# Cisco's TE++ Auto-Capacity Demonstration

We do this with a feature called “auto-capacity”.  You will find references to Container LSP or TE++ in other vendors’ documents, but the concept is the same. As traffic demands grow and shrink, we will use LSP splitting functionality to efficiently fill our total reservable bandwidth, i.e bin-packing the links.  

For this article, we run a customer demo using Cisco 8000 series routers which are powered by Silicon One ASICs. The topology is the following:

![mpls-te-lab-topology.png]({{site.baseurl}}/images/mpls-te-lab-topology.png)

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

Let us see it in action:
Sending ~200Mbps (on the interface and into tunnel Cisco) into the network:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#monitor interface for0/3/0/3 tunnel-te Cisco
Fri Aug 19 04:00:42.780 UTC
Cisco-8000                  Monitor Time: 00:00:24          SysUptime: 481:46:44
Protocol:General
Interface             In(bps)      Out(bps)     InBytes/Delta  OutBytes/Delta
Fo0/3/0/3              1000/  0%   196.0M/  0%   231.4M/176        7.5T/49.0M 
Cisco                     0/ --%   189.5M/ --%        0/0         30.2G/48.1M 
Quit='q',     Clear='c',    Freeze='f', Thaw='t',
Next set='n', Prev set='p', Bytes='y',  Packets='k'
(General='g', IPv4 Uni='4u', IPv4 Multi='4m', IPv6 Uni='6u', IPv6 Multi='6m')
</code>
</pre>
</div>

We do have a RVSP LSP built:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#sh rsvp reservation                         
Fri Aug 19 04:03:55.778 UTC
Destination Add DPort      Source Add SPort Pro   Input IF Sty Serv   Rate Burst
--------------- ----- --------------- ----- --- ---------- --- ---- ------ -----
   10.100.100.4 33231    10.100.100.1     9   0  Fo0/3/0/3  SE LOAD   169M    1K
RP/0/RP0/CPU0:Cisco-8000#
RP/0/RP0/CPU0:Cisco-8000#sh rsvp reservation detail                  
Fri Aug 19 04:06:14.844 UTC
RESV: IPv4-LSP Session addr: 10.100.100.4. TunID: 33231. LSPId: 9.
 Source addr: 10.100.100.1. ExtID: 10.100.100.1.
 Input adjusted interface: Fo0/3/0/3. Input physical interface: Fo0/3/0/3.
 Next hop: 10.100.14.4 (lih: 0x9).
 Style: Shared-Explicit. Service: Controlled-Load.
 Rate: 169110000 bits/sec. Burst: 1K bytes. Peak: 169110K bits/sec.
 MTU min: 40, max: 1500 bytes. 
 Flags: None.
 State expires in 359.188 sec.
 Policy:  Accepted. Policy source(s): Default.
 Header info: RSVP TTL=254. IP TTL=254. Flags: 0x1. TOS=0xc0.
 Resource: 
  Labels: Outgoing downstream: 3.
RP/0/RP0/CPU0:Cisco-8000#
</code>
</pre>
</div>

Let us look at the how many tunnels (should be one, i.e. <200Mbps of nominal/starting bandwidth).  Notice it is using the 1Gbps interface in the middle of our topology – Fo0/3/0/3:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#sh mpls traffic-eng tunnels signame 
Fri Aug 19 04:07:15.537 UTC
TunID:LSPID Ingress Intf:Label  Egress Intf:Label Bandwidth Tunnel Name         
----------- ------------------ ------------------ --------- --------------------
33231:9              N/A:N/A      Fo0/3/0/3:i-nul    169110 Cisco               
Displayed 1 (of 1) heads, 0 (of 0) midpoints, 0 (of 0) tails
Displayed 1 up, 0 down, 0 recovering, 0 recovered heads
RP/0/RP0/CPU0:Cisco-8000#
</code>
</pre>
</div>

Let us add 1Gbps of traffic to fill our reservable 1Gbps worth of bandwidth:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Cisco-8000                  Monitor Time: 00:01:46          SysUptime: 482:03:11

Protocol:General
Interface             In(bps)      Out(bps)     InBytes/Delta  OutBytes/Delta
Fo0/3/0/3              1000/  0%   979.9M/  2%   231.6M/94         7.5T/245.1M
Cisco                     0/ --%   216.9M/ --%        0/0         63.6G/0     

Quit='q',     Clear='c',    Freeze='f', Thaw='t',
Next set='n', Prev set='p', Bytes='y',  Packets='k'
(General='g', IPv4 Uni='4u', IPv4 Multi='4m', IPv6 Uni='6u', IPv6 Multi='6m')
</code>
</pre>
</div>

And we now check tunnels:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#sh mpls traffic-eng tunnels signame         
Fri Aug 19 04:17:59.483 UTC

TunID:LSPID Ingress Intf:Label  Egress Intf:Label Bandwidth Tunnel Name         
----------- ------------------ ------------------ --------- --------------------
33231:13             N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco               
33232:5              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-1             
33233:4              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-2             
33234:3              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-3             
33235:3              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-4  


RP/0/RP0/CPU0:Cisco-8000#sh rsvp reservation                         
Fri Aug 19 04:19:14.586 UTC
Destination Add DPort      Source Add SPort Pro   Input IF Sty Serv   Rate Burst
--------------- ----- --------------- ----- --- ---------- --- ---- ------ -----
   10.100.100.4 33231    10.100.100.1    13   0  Fo0/3/0/3  SE LOAD   150M    1K
   10.100.100.4 33232    10.100.100.1     5   0  Fo0/3/0/3  SE LOAD   150M    1K
   10.100.100.4 33233    10.100.100.1     4   0  Fo0/3/0/3  SE LOAD   150M    1K
   10.100.100.4 33234    10.100.100.1     3   0  Fo0/3/0/3  SE LOAD   150M    1K
   10.100.100.4 33235    10.100.100.1     3   0  Fo0/3/0/3  SE LOAD   150M    1K
   10.100.100.4 33236    10.100.100.1     4   0  Fo0/3/0/3  SE LOAD   150M    1K
   10.100.100.4 33237    10.100.100.1     3   0  Hu0/3/0/6  SE LOAD   150M    1K
   10.100.100.4 33238    10.100.100.1     2   0  Hu0/3/0/6  SE LOAD   150M    1K
   10.100.100.4 33239    10.100.100.1     2   0  Fo0/3/0/0  SE LOAD   150M    1K
</code>
</pre>
</div>

If you quickly do the math 5x150Mbps = 750Mbps as we continue packing this link.

Let us add 3Gbps of traffic to exhaust our present link capacity and start packing the other two links and you will see the following in a few min:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#sh mpls traffic-eng tunnels signame 
Fri Aug 19 04:39:16.315 UTC

TunID:LSPID Ingress Intf:Label  Egress Intf:Label Bandwidth Tunnel Name         
----------- ------------------ ------------------ --------- --------------------
33231:13             N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco               
33232:5              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-1             
33233:4              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-2             
33234:7              N/A:N/A      Hu0/3/0/6:24118    150000 Cisco-3             
33235:3              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-4             
33236:6              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-5             
33237:7              N/A:N/A      Hu0/3/0/6:24119    150000 Cisco-6             
33238:3              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-7             
33239:4              N/A:N/A      Hu0/3/0/6:24123    150000 Cisco-8             
33240:2              N/A:N/A      Hu0/3/0/6:24117    150000 Cisco-9             
33241:2              N/A:N/A      Fo0/3/0/0:24010    150000 Cisco-10            
33242:2              N/A:N/A      Hu0/3/0/6:24121    150000 Cisco-11            
33243:2              N/A:N/A      Hu0/3/0/6:24122    150000 Cisco-12            
33244:2              N/A:N/A      Hu0/3/0/6:24124    150000 Cisco-13            
33245:2              N/A:N/A      Fo0/3/0/0:24013    150000 Cisco-14            
33246:2              N/A:N/A      Fo0/3/0/0:24014    150000 Cisco-15            
Displayed 16 (of 16) heads, 0 (of 0) midpoints, 0 (of 0) tails
Displayed 16 up, 0 down, 0 recovering, 0 recovered heads
RP/0/RP0/CPU0:Cisco-8000#
</code>
</pre>
</div>

All 16 tunnels have been constructed and two other links are being used (Fo0/3/0/0 and Hu0/3/0/6).  If you wait, the bandwidth per tunnel will go up.  

Let us lower the traffic to 500Mbps to make sure we can merge the tunnels back.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#monitor interface for0/3/0/3 tunnel-te Cisco
Fri Aug 19 05:16:56.585 UTC
Cisco-8000                  Monitor Time: 00:00:36          SysUptime: 483:03:10
Protocol:General
Interface             In(bps)      Out(bps)     InBytes/Delta  OutBytes/Delta
Fo0/3/0/3              1000/  0%   490.1M/  1%   232.4M/88         7.8T/122.5M
Cisco                     0/ --%   223.0M/ --%        0/0         93.1G/0     
Quit='q',     Clear='c',    Freeze='f', Thaw='t',
Next set='n', Prev set='p', Bytes='y',  Packets='k'
(General='g', IPv4 Uni='4u', IPv4 Multi='4m', IPv6 Uni='6u', IPv6 Multi='6m’)q
</code>
</pre>
</div>

Our expectation is that after a few minutes we should be down to 3 tunnels (3x200Mbps since we are sending ~500Mbps into the tunnel):

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco-8000#sh mpls traffic-eng tunnels signame 
Fri Aug 19 05:15:30.784 UTC
TunID:LSPID Ingress Intf:Label  Egress Intf:Label Bandwidth Tunnel Name         
----------- ------------------ ------------------ --------- --------------------
33231:22             N/A:N/A      Fo0/3/0/3:i-nul    178350 Cisco               
33254:2              N/A:N/A      Fo0/3/0/3:i-nul    178350 Cisco-1             
33255:2              N/A:N/A      Fo0/3/0/3:i-nul    150000 Cisco-2             
Displayed 3 (of 3) heads, 0 (of 0) midpoints, 0 (of 0) tails
Displayed 3 up, 0 down, 0 recovering, 0 recovered heads
RP/0/RP0/CPU0:Cisco-8000#
</code>
</pre>
</div>

If you are interested in running these tests yourself, compare it to an example of a timeline we observed.

|     Time elapsed in min    |     Traffic rate      |     Tunnels    |
|----------------------------|-----------------------|----------------|
|     0                      |     1Gbps             |     1          |
|     +5                     |     1Gbps             |     5          |
|     +1                     |     3Gbps             |     5          |
|     +11                    |     3Gbps             |     14         |
|     +4                     |     3Gbps             |     16         |
|     +4                     |     0.5Gbps           |     16         |
|     +1                     |     0.5Gbps           |     14         |
|     +11                    |     0.5Gbps           |     4          |
|     +13                    |     0.5Gbps           |     3          |

# Conclusion

Cisco auto-capacity features handles bin-backing automatically without operator intervention. This article covered a practical example which was used for a customer demonstration.  Experiment with the timers that work for your network traffic demands. The configuration guide, available on cisco.com covers additional references: [Configuring Auto-Bandwidth TE++ on Cisco 8000](https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/mpls/710x/b-mpls-cg-cisco8000-710x/implementing-mpls-traffic-engineering.html#concept_jgv_qld_sxb)

# Acknowledgement

This article has been co-written with Thomas Peiyao Wang, Technical Marketing Engineer at Cisco. I'd like to thanks Thomas for his support during the lab tests.
