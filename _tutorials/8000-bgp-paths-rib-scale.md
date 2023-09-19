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
Paths: (1 available, best #1)
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
Paths: (2 available, best #1)
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
Paths: (2 available, best #1)
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

RP/0/RP0/CPU0:8201-32FH#sh bgp 1.0.0.0/24 bestpath-compare
Tue Sep 12 13:13:02.458 UTC
BGP routing table entry for 1.0.0.0/24
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker            16342698     16342698
    Flags: 0x00003001+0x20010000;
Last Modified: Sep 12 12:49:20.425 for 00:23:42
Paths: (24 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x3000000001060001+0x00, import: 0x020
  Not advertised to any peer
  64537 65010 65000 271253 3356 6762 13335, (aggregated by 13335 10.34.36.100)
    10.70.79.79 from 10.70.79.79 (13.37.79.79), if-handle 0x00000000
      Origin IGP, localpref 100, valid, external, best, group-best
      Received Path ID 0, Local Path ID 1, version 16342698
      Origin-AS validity: (disabled)
      best of AS 64537, Overall best
  Path #2: Received by speaker 0
  Flags: 0x3000000000020001+0x00, import: 0x020
  Not advertised to any peer
  64537 65011 65000 271253 3356 6762 13335, (aggregated by 13335 10.34.36.100)
    11.70.79.79 from 11.70.79.79 (13.37.79.79), if-handle 0x00000000
      Origin IGP, localpref 100, valid, external
      Received Path ID 0, Local Path ID 0, version 0
      Origin-AS validity: (disabled)
      Higher neighbor address than best path (path #1)
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
  Known via "bgp 65537", distance 20, metric 0
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

   via 10.70.79.79/32, 5 dependencies, recursive, bgp-ext [flags 0x6020]
    path-idx 0 NHID 0x0 [0x986d42a8 0x0], Internal 0xae0120a0
    next hop 10.70.79.79/32 via 10.70.79.79/32

    Load distribution: 0 (refcount 1)

    Hash  OK  Interface                 Address
    0     Y   HundredGigE0/0/0/5.10     10.70.79.79
</code>
</pre>
</div>

**Note:** The default behavior described here can change if BGP multipath or add-path feature is used, or if the device processes Internet table inside different VRFs. This is outside of this article scope.
{: .notice--primary}

# Scaling Cisco 8000 BGP table to 25M paths: practical demonstration
# Multiple Paths impact on RIB and FIB
# Going beyond the limits
# Monitoring
# Looking ahead: the future of BGP table scale