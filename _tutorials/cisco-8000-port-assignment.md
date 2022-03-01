---
published: true
date: '2022-03-01 22:07 +0100'
title: 'Port Assignments on Cisco 8100/8200 and Cisco 8800 Platforms '
position: hidden
tags:
  - iosxr
  - Cisco 8000
author: Chang Soo Lee
excerpt: >-
  This article will highlight how ports are allocated to IFG/Slice/NPU for each
  Cisco 8000 line card and systems.
---
## Introduction 

This memo will highlight how ports are allocated to IFG/Slice/NPU for each line card and system. Understanding port assignment will aid network design planning and, by enabling a deeper understanding of traffic flow, facilitate rapid, post-hoc system troubleshooting.  
Let’s review how this allocation is done, platform by platform and line card by line card. The following CLI is used to identify the port/IFG/Slice/NPU assignment: 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Cisco8201-32FH#sh controllers npu voq-usage interface all instance all loc 0/rp0/cpu0

-------------------------------------------------------------------
Node ID: 0/RP0/CPU0
Intf         Intf     <mark>NPU Slice IFG</mark>  Sys   VOQ   Flow   VOQ    Port
name         handle    <mark>#    #    #</mark>   Port  base  base   port   speed
             (hex)                                      type
----------------------------------------------------------------------
Hu0/0/0/0    f000110   0   4    1     4    448      0 local   100G
Hu0/0/0/1    f000118   0   4    1     8    456      0 local   100G
Hu0/0/0/2    f000120   0   4    1    12    464      0 local   100G
Hu0/0/0/3    f000128   0   4    0    16    480      0 local   100G
Hu0/0/0/4    f000130   0   3    1    20    496      0 local   100G
Hu0/0/0/5    f000138   0   4    0    24    488      0 local   100G
Hu0/0/0/6    f000140   0   5    0    28    512      0 local   100G
Hu0/0/0/7    f000148   0   5    1    32    528      0 local   100G
Hu0/0/0/8    f000150   0   5    1    36    536      0 local   100G
Hu0/0/0/9    f000158   0   5    1    40    544      0 local   100G
Hu0/0/0/10   f000160   0   5    0    44    520      0 local   100G
Hu0/0/0/11   f000168   0   5    0    48    560      0 local   100G
Hu0/0/0/12   f000170   0   3    0    52    576      0 local   100G
Hu0/0/0/13   f000178   0   3    1    56    504      0 local   100G
Hu0/0/0/14   f000180   0   3    0    60    584      0 local   100G
Hu0/0/0/15   f000188   0   3    0    64    592      0 local   100G
Hu0/0/0/16   f000190   0   2    1    68    608      0 local   100G
Hu0/0/0/17   f000198   0   2    1    72    616      0 local   100G
Hu0/0/0/18   f0001a0   0   2    1    76    624      0 local   100G
Hu0/0/0/19   f0001a8   0   2    0    80    640      0 local   100G
~ snip ~

</code>
</pre>
</div>


- NPU #: NPU number, should be 0 for 8100/8200
**Note** NPU number should be 0, 1, or 2 for 88-LC0-36FH line card (not shown above).  
NPU number should be should be 0, 1 for 88-LC0-34H14FH line card (not shown above)
{: .notice--primary}
- NPU core: Slice number, should be between 0 and 5 for 8201/8202/8201-32FH
- PP port: IFG number, should be 0 or 1 within slice 