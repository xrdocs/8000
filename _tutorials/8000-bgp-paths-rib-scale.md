---
published: true
date: '2023-09-19 15:54 +0200'
title: Scaling Cisco 8000 BGP table to 25M paths
author: Frederic Cuiller
position: hidden
tags:
  - iosxr
  - Cisco 8000
  - BGP table scale
  - BGP paths
excerpt: >-
  This article will demonstrate how Cisco 8000 Series routers can scale up to
  25M BGP paths.
---
# Introduction
It’s common for network operators Internet border routers to receive and process several full routing tables. They can then influence traffic leaving and entering their networks using route policies to manipulate BGP attributes.  

For such role, a recurrent question is how many full tables can be processed by a device? How many routes can be installed into RIB? How many BGP paths can the router hold? This article will answer this question for Cisco 8000 Series routers, illustrated with a practical demonstration.

# BGP table, RIB, FIB: the basics

There is still some confusion between BGP table, RIB, FIB and their associated scale; especially when several full tables are processed by a single device.  

Here is a quick refresh on the basics:

- Internet Border router receives first BGP table, and stores prefixes in BGP table along with all their attributes. There is 1 path per prefix:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp 1.0.0.0/24
Tue Sep 12 08:42:11.636 UTC
BGP routing table entry for 1.0.0.0/24
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                   3            3
Last Modified: Sep 12 08:28:58.425 for 00:13:13
<mark>Paths: (1 available</mark>, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Not advertised to any peer
  64537 65010 65000 264479 13335, (aggregated by 13335 10.34.36.100)
    10.70.79.79 from 10.70.79.79 (13.37.79.79)
      Origin IGP, localpref 100, valid, external, best, group-best
      Received Path ID 0, Local Path ID 1, version 3
      Origin-AS validity: (disabled)
</code>
</pre>
</div>

- Additional full tables are received from other BGP neighbors. The number of paths for the same prefix increases. Attributes could vary (different AS path, communities, etc.) for those extra paths:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp 1.0.0.0/24
Tue Sep 12 08:42:11.636 UTC
BGP routing table entry for 1.0.0.0/24
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                   3            3
Last Modified: Sep 12 08:28:58.425 for 00:13:13
<mark>Paths: (2 available</mark>, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Not advertised to any peer
  64537 65010 65000 264479 13335, (aggregated by 13335 10.34.36.100)
    10.70.79.79 from 10.70.79.79 (13.37.79.79)
      Origin IGP, localpref 100, valid, external, best, group-best
      Received Path ID 0, Local Path ID 1, version 3
      Origin-AS validity: (disabled)
  Path #2: Received by speaker 0
  Not advertised to any peer
  64537 65011 65000 264479 13335, (aggregated by 13335 10.34.36.100)
    11.70.79.79 from 11.70.79.79 (13.37.79.79)
      Origin IGP, localpref 100, valid, external
      Received Path ID 0, Local Path ID 0, version 0
      Origin-AS validity: (disabled)
  Path #3: Received by speaker 0
  Not advertised to any peer
</code>
</pre>
</div>

- BGP Best Path algorithm kicks in, and by default it selects 1 single path: it’s simply called the best path.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp 1.0.0.0/24
Tue Sep 12 08:42:11.636 UTC
BGP routing table entry for 1.0.0.0/24
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker                   3            3
Last Modified: Sep 12 08:28:58.425 for 00:13:13
<mark>Paths: (2 available, <b>best #1</b>)</mark>
  Not advertised to any peer
  <mark>Path #1:</mark> Received by speaker 0
  Not advertised to any peer
  64537 65010 65000 264479 13335, (aggregated by 13335 10.34.36.100)
    10.70.79.79 from 10.70.79.79 (13.37.79.79)
      Origin IGP, localpref 100, valid, external, best, group-best
      Received Path ID 0, Local Path ID 1, version 3
      Origin-AS validity: (disabled)
  Path #2: Received by speaker 0
  Not advertised to any peer
  64537 65011 65000 264479 13335, (aggregated by 13335 10.34.36.100)
    11.70.79.79 from 11.70.79.79 (13.37.79.79)
      Origin IGP, localpref 100, valid, external
      Received Path ID 0, Local Path ID 0, version 0
      Origin-AS validity: (disabled)
  Path #3: Received by speaker 0
  Not advertised to any peer

RP/0/RP0/CPU0:8201-32FH#sh bgp 1.0.0.0/24 <mark>bestpath-compare</mark>
Tue Sep 12 13:13:02.458 UTC
BGP routing table entry for 1.0.0.0/24
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker            16342698     16342698
    Flags: 0x00003001+0x20010000;
Last Modified: Sep 12 12:49:20.425 for 00:23:42
<mark>Paths: (24 available, best #1)</mark>
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x3000000001060001+0x00, import: 0x020
  Not advertised to any peer
  64537 65010 65000 271253 3356 6762 13335, (aggregated by 13335 10.34.36.100)
    10.70.79.79 from 10.70.79.79 (13.37.79.79), if-handle 0x00000000
      Origin IGP, localpref 100, valid, external, best, group-best
      Received Path ID 0, Local Path ID 1, version 16342698
      Origin-AS validity: (disabled)
      <mark>best of AS 64537, Overall best</mark>
  Path #2: Received by speaker 0
  Flags: 0x3000000000020001+0x00, import: 0x020
  Not advertised to any peer
  64537 65011 65000 271253 3356 6762 13335, (aggregated by 13335 10.34.36.100)
    11.70.79.79 from 11.70.79.79 (13.37.79.79), if-handle 0x00000000
      Origin IGP, localpref 100, valid, external
      Received Path ID 0, Local Path ID 0, version 0
      Origin-AS validity: (disabled)
      <mark>Higher neighbor address than best path (path #1)</mark>
-- snip ---
</code>
</pre>
</div>

- The BGP best path is then programmed into the routing table, often referred as RIB (Routing Information Base). RIB stores these routes and selects the best ones from among all other routing protocols. For Internet routes, they usually come from a single protocol: BGP.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh route 1.0.0.0/24
Tue Sep 12 08:47:13.672 UTC

Routing entry for 1.0.0.0/24
  <mark>Known via "bgp 65537", distance 20, metric 0</mark>
  Tag 64537, type external
  Installed Sep 12 08:46:38.354 for 00:00:35
  Routing Descriptor Blocks
    10.70.79.79, from 10.70.79.79, BGP external
      Route metric is 0
  No advertising protos.
</code>
</pre>
</div>

- Last, hardware must be programmed: this happens at Forwarding Information Base (FIB) level. In IOS XR, RIB downloads the set of selected best routes to the FIB processes using the Bulk Content Downloader (BCDL) process, onto each line card.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh cef 1.0.0.0/24
Tue Sep 12 12:16:48.986 UTC
1.0.0.0/24, version 589974355, internal 0x5000001 0x40 (ptr 0xdb5a2ba8) [1], 0x0 (0x0), 0x0 (0x0)
 Updated Sep 12 11:55:26.418
 Prefix Len 24, traffic index 0, precedence n/a, priority 4
  gateway array (0x976e97b8) reference count 997247, flags 0x2010, source rib (7), 0 backups
                [1 type 3 flags 0x48441 (0x6242e98) ext 0x0 (0x0)]
  LW-LDI[type=0, refc=0, ptr=0x0, sh-ldi=0x0]
  gateway array update type-time 1 Sep 12 08:29:06.295
 LDI Update time Sep 12 08:29:06.301

  Level 1 - Load distribution: 0
  [0] via 10.70.79.79/32, recursive

   <mark>via 10.70.79.79/32, 5 dependencies, recursive, bgp-ext [flags 0x6020]
    path-idx 0 NHID 0x0 [0x986d42a8 0x0], Internal 0xae0120a0
    next hop 10.70.79.79/32 via 10.70.79.79/32

    Load distribution: 0 (refcount 1)

    Hash  OK  Interface                 Address
    0     Y   HundredGigE0/0/0/5.10     10.70.79.79</mark>
</code>
</pre>
</div>

**Note:** The default behavior described here can change if BGP multipath or add-path feature is used, or if the device processes Internet table inside different VRFs. This is outside of this article scope.
{: .notice--info}

# Scaling Cisco 8000 BGP table to 25M paths: practical demonstration

This section demonstrates Cisco 8000 capability to scale to more than 20M BGP paths.  

To realize this test, a 8201-32FH router running IOS XR 7.9.2 is used as Device Under Test (DUT). 8201-32FH uses Intel Xeon D-1530 CPU @ 2.40GHz along with 32GB of RAM.  

24 x eBGP sessions are established to receive multiple copies of IPv4 global Internet routing table. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp sum
Wed Sep 13 13:04:40.082 UTC
BGP router identifier 13.37.1.1, local AS number 65537
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 551067265
BGP main routing table version 551067265
BGP NSR Initial initsync version 1025263 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker       551067265  551067265  551067265  551067265   551067265           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
1.63.51.21        0 65001       0       0        0    0    0 00:00:00 Active
10.70.79.79       0 65010 5280369    1719 551067265    0    0    1d04h    1010399
11.70.79.79       0 65011 5217380    1719 551067265    0    0    1d04h    1010399
12.70.79.79       0 65012 5205922    1719 551067265    0    0    1d04h    1010399
13.70.79.79       0 65013 5189440    1719 551067265    0    0    1d04h    1010399
14.70.79.79       0 65014 5196088    1719 551067265    0    0    1d04h    1010399
15.70.79.79       0 65015 5184072    1719 551067265    0    0    1d04h    1010399
16.70.79.79       0 65016 5187155    1719 551067265    0    0    1d04h    1010399
17.70.79.79       0 65017 5189922    1719 551067265    0    0    1d04h    1010399
18.70.79.79       0 65018 5197089    1719 551067265    0    0    1d04h    1010399
19.70.79.79       0 65019 5201890    1719 551067265    0    0    1d04h    1010399
20.70.79.79       0 65020 5201917    1719 551067265    0    0    1d04h    1010399
21.70.79.79       0 65021 5194267    1719 551067265    0    0    1d04h    1010399
22.70.79.79       0 65022 5194232    1719 551067265    0    0    1d04h    1010399
23.70.79.79       0 65023 5191012    1719 551067265    0    0    1d04h    1010399
24.70.79.79       0 65024 5193058    1719 551067265    0    0    1d04h    1010399
25.70.79.79       0 65025 5192615    1719 551067265    0    0    1d04h    1010399
26.70.79.79       0 65026 5197328    1719 551067265    0    0    1d04h    1010399
27.70.79.79       0 65027 5209301    1721 551067265    0    0    1d04h    1010399
28.70.79.79       0 65028 5191256    1719 551067265    0    0    1d04h    1010399
29.70.79.79       0 65029 5216363    1719 551067265    0    0    1d04h    1010399
30.70.79.79       0 65030 5192554    1723 551067265    0    0    1d04h    1010399
31.70.79.79       0 65031 5198880    1719 551067265    0    0    1d04h    1010399
32.70.79.79       0 65032 5192662    1719 551067265    0    0    1d04h    1010399
33.70.79.79       0 65033 5187821    1719 551067265    0    0    1d04h    1010399
</code>
</pre>
</div>

And similarly for IPv6, 24 x eBGP sessions are setup:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp ipv6 unicast summary
Tue Sep 12 12:35:46.551 UTC
BGP router identifier 13.37.1.1, local AS number 65537
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 6546135
BGP main routing table version 6546135
BGP NSR Initial initsync version 165973 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         6546135    6546135    6546135    6546135     6546135           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:db8:10:7079::79
                  0 65010 5597166    1722 497765891    0    0    1d04h     186594
2001:db8:11:7079::79
                  0 65011 5597049    1722 497765891    0    0    1d04h     186594
2001:db8:12:7079::79
                  0 65012 5597016    1722 497765891    0    0    1d04h     186594
2001:db8:13:7079::79
                  0 65013 5601651    1722 497765891    0    0    1d04h     186594
2001:db8:14:7079::79
                  0 65014 5599112    1722 497765891    0    0    1d04h     186594
2001:db8:15:7079::79
                  0 65015 5588400    1722 497765891    0    0    1d04h     186594
2001:db8:16:7079::79
                  0 65016 5598537    1722 497765891    0    0    1d04h     186594
2001:db8:17:7079::79
                  0 65017 5601125    1722 497765891    0    0    1d04h     186594
2001:db8:18:7079::79
                  0 65018 5576339    1722 497765891    0    0    1d04h     186594
2001:db8:19:7079::79
                  0 65019 5601651    1722 497765891    0    0    1d04h     186594
2001:db8:20:7079::79
                  0 65020 5582612    1722 497765891    0    0    1d04h     186594
2001:db8:21:7079::79
                  0 65021 5591860    1722 497765891    0    0    1d04h     186594
2001:db8:22:7079::79
                  0 65022 5581286    1722 497765891    0    0    1d04h     186594
2001:db8:23:7079::79
                  0 65023 5598931    1722 497765891    0    0    1d04h     186594
2001:db8:24:7079::79
                  0 65024 5582482    1722 497765891    0    0    1d04h     186594
2001:db8:25:7079::79
                  0 65025 5584450    1722 497765891    0    0    1d04h     186594
2001:db8:26:7079::79
                  0 65026 5598266    1722 497765891    0    0    1d04h     186594
2001:db8:27:7079::79
                  0 65027 5600159    1722 497765891    0    0    1d04h     186594
2001:db8:28:7079::79
                  0 65028 5592619    1722 497765891    0    0    1d04h     186594
2001:db8:29:7079::79
                  0 65029 5596220    1722 497765891    0    0    1d04h     186594
2001:db8:30:7079::79
                  0 65030 5581874    1722 497765891    0    0    1d04h     186594
2001:db8:31:7079::79
                  0 65031 5600379    1722 497765891    0    0    1d04h     186594
2001:db8:32:7079::79
                  0 65032 5590597    1722 497765891    0    0    1d04h     186594
2001:db8:33:7079::79
                  0 65033 5600118    1722 497765891    0    0    1d04h     186594
2001:db8:1337:0:1:63:51:21
                  0 65001       0       0        0    0    0 00:00:00 Active
</code>
</pre>
</div>

This represents a total of ~ 28M BGP paths spread across 48 x eBGP sessions:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp scale
Wed Sep 13 13:07:29.428 UTC

VRF: default
 Neighbors Configured: 50     <mark>Established: 48</mark>

 Address-Family   Prefixes Paths    PathElem   Prefix     Path       PathElem
                                               Memory     Memory     Memory
  IPv4 Unicast    1017385  24373748 1017385    143.60MB   2.00GB     103.82MB
  IPv6 Unicast    188244   4486597  188244     28.72MB    376.53MB   19.21MB
  ------------------------------------------------------------------------------
  <mark>Total           1205629  </mark>28860345 1205629    172.32MB   2.37GB     123.03MB

Total VRFs Configured: 0
</code>
</pre>
</div>

Going in details, each IPv4 prefix has 24 paths:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp
Tue Sep 12 13:29:28.013 UTC
BGP router identifier 13.37.1.1, local AS number 65537
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 18150169
BGP main routing table version 18150169
BGP NSR Initial initsync version 1025263 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
*> 0.0.0.0/0          10.70.79.79                            0 64537 65010 65000 48571 12302 i
*                     11.70.79.79                            0 64537 65011 65000 48571 12302 i
*                     12.70.79.79                            0 64537 65012 65000 48571 12302 i
*                     13.70.79.79                            0 64537 65013 65000 48571 12302 i
*                     14.70.79.79                            0 64537 65014 65000 48571 12302 i
*                     15.70.79.79                            0 64537 65015 65000 48571 12302 i
*                     16.70.79.79                            0 64537 65016 65000 48571 12302 i
*                     17.70.79.79                            0 64537 65017 65000 48571 12302 i
*                     18.70.79.79                            0 64537 65018 65000 48571 12302 i
*                     19.70.79.79                            0 64537 65019 65000 48571 12302 i
*                     20.70.79.79                            0 64537 65020 65000 48571 12302 i
*                     21.70.79.79                            0 64537 65021 65000 48571 12302 i
*                     22.70.79.79                            0 64537 65022 65000 48571 12302 i
*                     23.70.79.79                            0 64537 65023 65000 48571 12302 i
*                     24.70.79.79                            0 64537 65024 65000 48571 12302 i
*                     25.70.79.79                            0 64537 65025 65000 48571 12302 i
*                     26.70.79.79                            0 64537 65026 65000 48571 12302 i
*                     27.70.79.79                            0 64537 65027 65000 48571 12302 i
*                     28.70.79.79                            0 64537 65028 65000 48571 12302 i
*                     29.70.79.79                            0 64537 65029 65000 48571 12302 i
*                     30.70.79.79                            0 64537 65030 65000 48571 12302 i
*                     31.70.79.79                            0 64537 65031 65000 48571 12302 i
*                     32.70.79.79                            0 64537 65032 65000 48571 12302 i
*                     33.70.79.79                            0 64537 65033 65000 48571 12302 i
*> 1.0.0.0/24         10.70.79.79                            0 64537 65010 65000 271253 262691 13335 i
*                     11.70.79.79                            0 64537 65011 65000 271253 262691 13335 i
*                     12.70.79.79                            0 64537 65012 65000 271253 262691 13335 i
*                     13.70.79.79                            0 64537 65013 65000 271253 262691 13335 i
*                     14.70.79.79                            0 64537 65014 65000 271253 262691 13335 i
*                     15.70.79.79                            0 64537 65015 65000 271253 262691 13335 i
*                     16.70.79.79                            0 64537 65016 65000 271253 262691 13335 i
*                     17.70.79.79                            0 64537 65017 65000 271253 262691 13335 i
*                     18.70.79.79                            0 64537 65018 65000 271253 262691 13335 i
*                     19.70.79.79                            0 64537 65019 65000 271253 262691 13335 i
*                     20.70.79.79                            0 64537 65020 65000 271253 262691 13335 i
*                     21.70.79.79                            0 64537 65021 65000 271253 262691 13335 i
*                     22.70.79.79                            0 64537 65022 65000 271253 262691 13335 i
*                     23.70.79.79                            0 64537 65023 65000 271253 262691 13335 i
*                     24.70.79.79                            0 64537 65024 65000 271253 262691 13335 i
*                     25.70.79.79                            0 64537 65025 65000 271253 262691 13335 i
*                     26.70.79.79                            0 64537 65026 65000 271253 262691 13335 i
*                     27.70.79.79                            0 64537 65027 65000 271253 262691 13335 i
*                     28.70.79.79                            0 64537 65028 65000 271253 262691 13335 i
*                     29.70.79.79                            0 64537 65029 65000 271253 262691 13335 i
*                     30.70.79.79                            0 64537 65030 65000 271253 262691 13335 i
*                     31.70.79.79                            0 64537 65031 65000 271253 262691 13335 i
*                     32.70.79.79                            0 64537 65032 65000 271253 262691 13335 i
*                     33.70.79.79                            0 64537 65033 65000 271253 262691 13335 i
-- snip --
</code>
</pre>
</div>

And the same for IPv6 prefixes:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp ipv6 unicast dfa-regex  _109_
Tue Sep 12 13:43:11.290 UTC
BGP router identifier 13.37.1.1, local AS number 65537
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 8656025
BGP main routing table version 8656025
BGP NSR Initial initsync version 165973 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
*> 2001:420::/32      2001:db8:10:7079::79
                                                             0 64537 65010 65000 199524 6939 109 i
*                     2001:db8:11:7079::79
                                                             0 64537 65011 65000 199524 6939 109 i
*                     2001:db8:12:7079::79
                                                             0 64537 65012 65000 199524 6939 109 i
*                     2001:db8:13:7079::79
                                                             0 64537 65013 65000 199524 6939 109 i
*                     2001:db8:14:7079::79
                                                             0 64537 65014 65000 199524 6939 109 i
*                     2001:db8:15:7079::79
                                                             0 64537 65015 65000 199524 6939 109 i
*                     2001:db8:16:7079::79
                                                             0 64537 65016 65000 199524 6939 109 i
*                     2001:db8:17:7079::79
                                                             0 64537 65017 65000 199524 6939 109 i
*                     2001:db8:18:7079::79
                                                             0 64537 65018 65000 199524 6939 109 i
*                     2001:db8:19:7079::79
                                                             0 64537 65019 65000 199524 6939 109 i
*                     2001:db8:20:7079::79
                                                             0 64537 65020 65000 199524 6939 109 i
*                     2001:db8:21:7079::79
                                                             0 64537 65021 65000 199524 6939 109 i
*                     2001:db8:22:7079::79
                                                             0 64537 65022 65000 199524 6939 109 i
*                     2001:db8:23:7079::79
                                                             0 64537 65023 65000 199524 6939 109 i
*                     2001:db8:24:7079::79
                                                             0 64537 65024 65000 199524 6939 109 i
*                     2001:db8:25:7079::79
                                                             0 64537 65025 65000 199524 6939 109 i
*                     2001:db8:26:7079::79
                                                             0 64537 65026 65000 199524 6939 109 i
*                     2001:db8:27:7079::79
                                                             0 64537 65027 65000 199524 6939 109 i
*                     2001:db8:28:7079::79
                                                             0 64537 65028 65000 199524 6939 109 i
*                     2001:db8:29:7079::79
                                                             0 64537 65029 65000 199524 6939 109 i
*                     2001:db8:30:7079::79
                                                             0 64537 65030 65000 199524 6939 109 i
*                     2001:db8:31:7079::79
                                                             0 64537 65031 65000 199524 6939 109 i
*                     2001:db8:32:7079::79
                                                             0 64537 65032 65000 199524 6939 109 i
*                     2001:db8:33:7079::79
                                                             0 64537 65033 65000 199524 6939 109 i
</code>
</pre>
</div>

As described in the introduction, a single path is programmed into RIB:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh route
Tue Sep 12 13:57:17.681 UTC

Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, t - Traffic Engineering, (!) - FRR Backup path

Gateway of last resort is 10.70.79.79 to network 0.0.0.0

B*   0.0.0.0/0 [20/0] via 10.70.79.79, 00:00:02
B    1.0.0.0/24 [20/0] via 10.70.79.79, 00:01:06
B    1.0.4.0/22 [20/0] via 10.70.79.79, 00:01:06
B    1.0.5.0/24 [20/0] via 10.70.79.79, 00:01:06
B    1.0.16.0/24 [20/0] via 10.70.79.79, 00:01:06
B    1.0.32.0/24 [20/0] via 10.70.79.79, 00:01:06
B    1.0.64.0/18 [20/0] via 10.70.79.79, 00:01:06
B    1.0.128.0/17 [20/0] via 10.70.79.79, 00:01:06
B    1.0.128.0/18 [20/0] via 10.70.79.79, 00:01:06
B    1.0.128.0/19 [20/0] via 10.70.79.79, 00:01:06
B    1.0.128.0/24 [20/0] via 10.70.79.79, 00:01:06
B    1.0.129.0/24 [20/0] via 10.70.79.79, 00:01:06
B    1.0.130.0/23 [20/0] via 10.70.79.79, 00:01:06
B    1.0.132.0/24 [20/0] via 10.70.79.79, 00:01:06
-- snip --
RP/0/RP0/CPU0:8201-32FH#sh route ipv4 unicast summary
Tue Sep 12 13:44:10.178 UTC
Route Source                     Routes     Backup     Deleted     Memory(bytes)
connected                        53         1          0           11664
local                            54         0          0           11664
application fib_mgr              0          0          0           0
ospf 1                           0          0          0           0
<mark>bgp 65537                        1001184</mark>    0          0           216255744
static                           0          0          0           0
dagr                             0          0          0           0
vxlan                            0          0          0           0
isis CORE                        0          0          0           0
Total                            1001291    1          0           216279072
</code>
</pre>
</div>

And the same for IPv6:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh route ipv6 unicast
Tue Sep 12 13:57:53.114 UTC

Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, t - Traffic Engineering, (!) - FRR Backup path

Gateway of last resort is fe80::46b6:beff:fe46:22d0 to network ::

B*   ::/0
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:33, HundredGigE0/0/0/5.10
B    1::1/128
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    2::1/128
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    10:1:2::/64
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    10:1:5::/64
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    11:1:2::/64
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    12::/64
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    12:1:2::/64
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    40::/11
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    60::/14
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    64::/17
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    64:8000::/18
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
B    64:c000::/19
      [20/0] via fe80::46b6:beff:fe46:22d0, 00:01:43, HundredGigE0/0/0/5.10
-- snip --
RP/0/RP0/CPU0:8201-32FH#sh route ipv6 unicast summary
Wed Sep 13 13:08:27.789 UTC
Route Source                     Routes     Backup     Deleted     Memory(bytes)
local-iid sidmgr                 0          0          0           0
local                            52         0          0           11232
connected                        52         0          0           11232
connected l2tpv3_xconnect        0          0          0           0
local-srv6 xtc_srv6              0          0          0           0
local-srv6 bgp-65537             0          0          0           0
local-srv6 isis-CORE             0          0          0           0
<mark>bgp 65537                        190957</mark>     0          0           216614392
static                           0          0          0           0
vxlan                            0          0          0           0
isis CORE                        0          0          0           0
Total                            191061     0          0           216636856
</code>
</pre>
</div>

**Note:** this router processes a real and live BGP feed. Therefore, it’s expected to see churn and the number of routes might slightly differ between outputs.
{: .notice--info}

FIB scale is similar to overall number of prefixes:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh cef summary
Wed Sep 13 13:10:38.537 UTC

Router ID is 192.168.0.23

IP CEF with switching (Table Version 0) for node0_RP0_CPU0

  Load balancing: L4
  Tableid 0xe0000000 (0x975b57e8), Vrfid 0x60000000, Vrid 0x20000000, Flags 0x1019
  Vrfname default, Refcount 1017634
  <mark>1017271 routes</mark>, 0 protected, 0 reresolve, 0 unresolved (0 old, 0 new), 219730536 bytes
    1017090 rib, 0 lsd, 71 aib, 0 internal, 106 interface, 4 special, 1 default routes
    Prefix masklen distribution:
        unicast: 13797 /32, 262 /31, 1330 /30, 1948 /29, 1399 /28, 1060 /27
                 934 /26, 1169 /25, 604749 /24, 108618 /23, 114766 /22, 54300 /21
                 45828 /20, 26009 /19, 14331 /18, 8490 /17, 13770 /16, 2143 /15
                 1211 /14, 576 /13, 297 /12, 103 /11, 40 /10, 14 /9
                 16 /8 , 1 /0

RP/0/RP0/CPU0:8201-32FH#sh cef ipv6 summary
Wed Sep 13 13:10:44.048 UTC

Router ID is 192.168.0.23

IP CEF with switching (Table Version 0) for node0_RP0_CPU0

  Load balancing: L4
  Tableid 0xe0800000 (0x97f85b90), Vrfid 0x60000000, Vrid 0x20000000, Flags 0x1019
  Vrfname default, Refcount 165786
  <mark>165554 routes</mark>, 0 protected, 0 reresolve, 0 unresolved (0 old, 0 new), 35759664 bytes
    165482 rib, 0 lsd, 65 aib, 0 internal, 0 interface, 6 special, 1 default routes
    Prefix masklen distribution:
        unicast: 323 /128, 250 /127, 135 /126, 6 /125, 8 /123, 9 /112
                 1 /96 , 1 /95 , 1 /94 , 1 /93 , 1 /92 , 1 /91
                 1 /90 , 1 /89 , 1 /88 , 1 /87 , 1 /86 , 1 /85
                 1 /84 , 1 /83 , 1 /82 , 1 /81 , 1 /80 , 1 /79
                 1 /78 , 1 /77 , 1 /76 , 1 /75 , 1 /74 , 1 /73
                 1 /72 , 1 /71 , 1 /70 , 1 /69 , 1 /68 , 1 /67
                 1 /66 , 3 /65 , 1199 /64 , 6 /63 , 3 /62 , 5 /61
                 7 /60 , 4 /59 , 8 /58 , 4 /57 , 682 /56 , 22 /55
                 162 /54 , 7 /53 , 35 /52 , 165 /51 , 28 /50 , 28 /49
                 56339 /48 , 4169 /47 , 3979 /46 , 2922 /45 , 14303 /44 , 2451 /43
                 3597 /42 , 1020 /41 , 12737 /40 , 1765 /39 , 1839 /38 , 1382 /37
                 5114 /36 , 1148 /35 , 3249 /34 , 3315 /33 , 18188 /32 , 3482 /31
                 3339 /30 , 5731 /29 , 6122 /28 , 5386 /27 , 582 /26 , 3 /25
                 96 /24 , 4 /23 , 37 /22 , 2 /21 , 55 /20 , 25 /19
                 11 /18 , 4 /17 , 2 /15 , 3 /14 , 4 /13 , 2 /12
                 1 /11 , 1 /10 , 1 /9  , 2 /8  , 2 /7  , 1 /6
                 1 /5  , 2 /4  , 1 /0
</code>
</pre>
</div>

All those prefixes are ultimately programmed into Silicon One LPM database:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh controllers npu resources lpmtcam location 0/RP0/CPU0
Wed Sep 13 13:12:22.068 UTC
HW Resource Information
    Name                            : lpm_tcam
    Asic Type                       : Q200

NPU-0
OOR Summary
        Estimated Max Entries       : 100
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


OFA Table Information
(May not match HW usage)
        iprte                       : 1017390
        ip6rte                      : 184229
        ip6mcrte                    : 0
        ipmcrte                     : 0

Current Hardware Usage
    Name: lpm_tcam
        Estimated Max Entries       : 100
        Total In-Use                : 38       (38 %)
        OOR State                   : Green

<mark>
       Name: v4_lpm
           Total In-Use                : 1017528


       Name: v6_lpm
           Total In-Use                : 184252
</mark>
</code>
</pre>
</div>

Let’s now check impact on memory. Following snapshot shows BGP process uses 6.8GB of RAM with such scale:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh processes memory detail
Wed Sep 13 15:06:50.245 UTC
JID         Text       Data       Stack      Dynamic    Dyn-Limit  Shm-Tot    Phy-Tot               Process
============================================================================================================
1089           2M      6820M       132K      6763M      7447M        77M      <mark>6817M bgp</mark>
194          620K      4521M       132K      3452M     22528M      1376M      4332M npu_drvr
1169           1M       710M       132K       588M      8192M       126M       714M ipv4_rib
-- snip –
</code>
</pre>
</div>

With overall memory utilization still in acceptable range (56%):

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh memory summary
Wed Sep 13 15:08:34.147 UTC

node:      node0_RP0_CPU0
------------------------------------------------------------------

 <mark>Physical Memory: 31595M total (14122M available)</mark>
 Application Memory : 31595M (14122M available)
 Image: 4M (bootram: 0M)
 Reserved: 0M, IOMem: 0M, flashfsys: 0M
 Total shared window: 2G
RP/0/RP0/CPU0:8201-32FH#
</code>
</pre>
</div>

# Multiple Paths impact on RIB and FIB


# Going beyond the limits

Cisco 8000 is currently certified for a maximum number of 20M BGP paths. While the test performed earlier shows it’s possible to go higher, the platform will hit limits after a certain scale.  

There are multiple factors limiting this scale:
- Hardware: the CPU & amount of memory used in the platform
- But also software: what’s the maximum amount of memory BGP process can allocate?

While 8201-32FH ships with 32GB of RAM, IOS XR will restrict the BGP process to 8GB. This is called the RLIMIT (Resource Limit). 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH#sh bgp process
Tue Sep 12 12:49:13.450 UTC

BGP Process Information:
BGP is operating in STANDALONE mode
Autonomous System number format: ASPLAIN
Autonomous System: 65537
Router ID: 13.37.1.1 (manually configured)
Default Cluster ID: 13.37.1.1
Active Cluster IDs:  13.37.1.1
Fast external fallover enabled
Platform Loadbalance paths max: 1024
<mark>Platform RLIMIT max: 8589934592 bytes</mark>
-- snip --
</code>
</pre>
</div>

**Info:** RLIMIT is platform dependent. It’s possible to observe some IOS XR based platforms with higher values (e.g 80GB for IOS XRv9000 virtual route-reflector platform).
{: .notice--info}

Adding additional neighbors will ultimately trigger memory allocation failure, causing BGP process errors or crash.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Sep 12 07:54:29.477 UTC: bgp[1089]: %ROUTING-BGP-3-NOMEM_RESET : [10420] : Failed to allocate memory for path, resetting neighbor  : bgp : (PID=10384) :  -Traceback= 55f00d13d929 7f67254aedb1 7f67254b8c44 7f6725db87be 55f00d13cfaf
RP/0/RP0/CPU0:Sep 12 07:54:29.477 UTC: bgp[1089]: %ROUTING-BGP-5-ADJCHANGE : neighbor 20.70.79.79 Down - No memory (VRF: default) (AS: 65020)
RP/0/RP0/CPU0:Sep 12 07:54:29.477 UTC: bgp[1089]: %ROUTING-BGP-5-ADJCHANGE : neighbor 38.70.79.79 Down - No memory (VRF: default) (AS: 65038)
RP/0/RP0/CPU0:Sep 12 07:54:33.922 UTC: bgp[1089]: %ROUTING-BGP-3-NOMSGCHUNK : [10418] : Failed to allocate 414 bytes from the memory pool for message of type 2  : bgp : (PID=10384) :  -Traceback= 55f00d188db2 7f67257412aa 7f67254aedb1 7f67254b8c44 7f6725db87be 55f00d18a30e
</code>
</pre>
</div>

# Monitoring

Several YANG models are available to monitor BGP process health, statistics, but also platform memory utilization:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Cisco-IOS-XR-ipv4-bgp-oper:bgp/bpm-instances-table/bpm-instances
Cisco-IOS-XR-nto-misc-oper:memory-summary/nodes/node/detail
Cisco-IOS-XR-procmem-oper:processes-memory/nodes/node/process-ids/process-id
</code>
</pre>
</div>

There are also OpenConfig models available to monitor the number of BGP paths:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Cisco-IOS-XR-ipv4-bgp-oc-oper:oc-bgp/bgp-rib/afi-safi-table/ipv4-unicast/loc-rib/num-routes/num-routes
Cisco-IOS-XR-ipv4-bgp-oc-oper:oc-bgp/bgp-rib/afi-safi-table/ipv6-unicast/loc-rib/num-routes/num-routes
</code>
</pre>
</div>


Here are sample dashboards which can be used:

![8000-bgp-memory.png]({{site.baseurl}}/images/8000-bgp-memory.png)

![8000-bgp-paths.png]({{site.baseurl}}/images/8000-bgp-paths.png)

# Looking ahead: the future of BGP table scale

This article demonstrated how Cisco 8000 could scale up to 25M+ BGP paths. It also went through some IOS XR BGP implementation details.  

Is this number good enough? Are 20M BGP paths sufficient for a peering router? Is it common for network operators to process 20+ full BGP feeds on a single device?  

As usual, the answer is: it depends. Different customers have diverse designs, use different features, and have different scale requirements. So far, only few hyperscaler customers expressed to support more.  

While BGP table continues to grow, there is still room for margin. If current maximum scale is not high enough, network operators can still rely on [IOS XR BGP Multi-Instance](https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/bgp/75x/b-bgp-cg-8k-75x/implementing-bgp.html#concept_qkq_mhc_qjb) feature: up to 4 BGP processes can be spawned, and address families can be spread across them to reach theoretically up to 80M paths.  

Last, recent routers have plenty of RAM available: most current Cisco 8000 have 32GB of RAM and new products now ship with 64GB of RAM. Currently, IOS XR doesn’t keep advantage of all this memory. There is plan in progress to increase BGP process RLIMIT, with a recent prototype IOS XR image demonstrating support of 75M BGP paths on a 8202-32FH-M with 64GB of RAM. This was possible using a 24GB RLIMIT value for the BGP process during a Customer Proof of Concept.

