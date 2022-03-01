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
{% include toc icon="table" title="Port Assignments on Cisco 8100/8200 and Cisco 8800 Platforms"%} 


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


- NPU #: NPU number  
-- NPU number should be 0 for 8100/8200 (8000 fixed systems)  
-- NPU number should be 0, 1, or 2 for 88-LC0-36FH line card (not shown above).  
-- NPU number should be should be 0, 1 for 88-LC0-34H14FH line card (not shown above)
- NPU core: Slice number, should be between 0 and 5 for 8201/8202/8201-32FH
- PP port: IFG number, should be 0 or 1 within slice

## Port allocation
### Cisco 8201
Cisco 8201 is made of a single Q100 NPU. 

![8201-ports.png]({{site.baseurl}}/images/8201-ports.png)

The table below shows port assignment to different slices and IFG:

| Interface  | NPU/Slice/IFG |
|------------|---------------|
| FH0/0/0/0  | 0/0/0         |
| FH0/0/0/1  | 0/0/0         |
| FH0/0/0/2  | 0/0/1         |
| FH0/0/0/3  | 0/0/1         |
| FH0/0/0/4  | 0/5/10        |
| FH0/0/0/5  | 0/5/10        |
| FH0/0/0/6  | 0/5/11        |
| FH0/0/0/7  | 0/5/11        |
| FH0/0/0/8  | 0/4/9         |
| FH0/0/0/9  | 0/4/9         |
| FH0/0/0/10 | 0/4/8         |
| FH0/0/0/11 | 0/4/8         |
| FH0/0/0/12 | 0/3/7         |
| FH0/0/0/13 | 0/3/7         |
| FH0/0/0/14 | 0/3/6         |
| FH0/0/0/15 | 0/3/6         |
| FH0/0/0/16 | 0/2/5         |
| FH0/0/0/17 | 0/2/5         |
| FH0/0/0/18 | 0/2/4         |
| FH0/0/0/19 | 0/2/4         |
| FH0/0/0/20 | 0/1/3         |
| FH0/0/0/21 | 0/1/3         |
| FH0/0/0/22 | 0/1/2         |
| FH0/0/0/23 | 0/1/2         |
| Hu0/0/0/24 | 0/3/7         |
| Hu0/0/0/25 | 0/3/6         |
| Hu0/0/0/26 | 0/2/5         |
| Hu0/0/0/27 | 0/2/4         |
| Hu0/0/0/28 | 0/1/3         |
| Hu0/0/0/29 | 0/1/2         |
| Hu0/0/0/30 | 0/0/1         |
| Hu0/0/0/31 | 0/0/0         |
| Hu0/0/0/32 | 0/5/11        |
| Hu0/0/0/33 | 0/5/10        |
| Hu0/0/0/34 | 0/4/9         |
| Hu0/0/0/35 | 0/4/8         |

### Cisco 8202
Cisco 8202 is made of a single Q100 NPU.  
![8202.png]({{site.baseurl}}/images/8202.png)


### Cisco 8201-32FH
Cisco 8201-32FH is made of a single Q200 NPU.  
![8201-32FH.png]({{site.baseurl}}/images/8201-32FH.png)


### Cisco 8201-24H8FH
Cisco 8201-24H8FH is made of a single Q200 NPU.  
![8201-24H8FH.png]({{site.baseurl}}/images/8201-24H8FH.png)


### Cisco 8101-32FH
Cisco 8101-32FH is made of a single Q200L NPU. 
![8101-32FH.png]({{site.baseurl}}/images/8101-32FH.png)


### Cisco 8101-32H
Cisco 8101-32H is made of a single Q202L NPU. 
![8101-32H.png]({{site.baseurl}}/images/8101-32H.png)


### Cisco 8102-64H
Cisco 8102-64H is made of a single Q201L NPU.  
![8102-64H.png]({{site.baseurl}}/images/8102-64H.png)


### 8800-LC-36FH
There are four Q100 NPUs on these line cards.  
![8800-LC-36FH.png]({{site.baseurl}}/images/8800-LC-36FH.png)


### 88-LC0-36FH-M and 88-LC0-36FH
There are three Q200 NPUs on these line cards.  
![88-LC0-36FH-M.png]({{site.baseurl}}/images/88-LC0-36FH-M.png)

![88-LC0-36FH.png]({{site.baseurl}}/images/88-LC0-36FH.png)

88-LC0-36FH-M and 88-LC0-36FH are identical in all ways except the former can support MACSec and the latter cannot.

### 88-LC0-34H14FH
There are two Q200 NPUs on these line cards.  
![88-LC0-34H14FH.png]({{site.baseurl}}/images/88-LC0-34H14FH.png)

