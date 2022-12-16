---
published: true
date: '2022-12-13 18:14 +0100'
title: Cisco 8000 PBTS Implementation (Basic)
author: Yosef Manasseh
excerpt: >-
  This post aims to describe PBTS implementation on Cisco 8000 routers.   It
  will go over feature operation, configuration, verification, and other notes
  like constraints and supported scale.
tags:
  - iosxr
  - Cisco 8000
  - PBTS
  - MPLS-TE
position: hidden
---
{% include toc icon="table" title="Cisco 8000 PBTS Implementation (Basic)" %}

## Introduction
This post aims to describe PBTS implementation on Cisco 8000 routers.  
It will go over feature operation, configuration, verification, and other notes like constraints and supported scale.

## Feature Description
**PBTS (Policy Based Tunnel Selection)** is a feature where operator can classify incoming network traffic and forward that traffic to destination via specific MPLS traffic engineering tunnels (TE tunnel).

PBTS implementation on Cisco 8000 described in this article classifies the incoming traffic on ingress interface based on `EXP` field (for MPLS traffic) or `prec/DSCP` field (for IP traffic).  
Thus it is possible, for instance, to forward traffic with specific EXP/Prec/DSCP value via a specific TE tunnel like shown in below picture.

![01_overview.png]({{site.baseurl}}/images/01_overview.png)  

---

In Cisco 8000 platform, PBTS is implemented in following manner:

1. Plain IPv4 / plain IPv6 / MPLS **traffic arrives** on ingress interface.

2. Using **class-map** inside **policy-map** set in inbound direction on **ingress interface**, **traffic is classified** based on MPLS EXP or IP prec/DSCP fields and assigned to specific **forward-class (FC)**.

3. Each TE tunnel is associated with a specific FC.  
In this way, specific traffic flow can then be **steered to TE tunnel** with the matching FC.  
e.g.  
Traffic with IP prec_6 ➜ FC 3 ➜ tunnel-te1  

---

PBTS support on 8000 platform comes in **phases**.  
The following constraints apply for phase 1 in IOS XR 7.5.3:

- [Supported](#link){: .btn .btn--success} PBTS over TE tunnel and also LDP-over-TE tunnel.

- [Supported](#link){: .btn .btn--success} PBTS for IPv4, IPv6, MPLS traffic.

- [NOT Supported](#link){: .btn .btn--danger} PBTS for services (L3VPN, 6VPE, 6PE, L2VPN) and overlays over this prefix over TE tunnel with PBTS enabled.

- [NOT Supported](#link){: .btn .btn--danger} Statically configured MPLS prefix over LDP-over-TE tunnel.

- [NOT Supported](#link){: .btn .btn--danger} Statically configured MPLS prefix over TE tunnel.

- [NOT Supported](#link){: .btn .btn--danger} Labeled Unicast over TE tunnel.

The [NOT Supported](#link){: .btn .btn--danger} items will be addressed in future phase in later IOS XR release.

---

Some important **notes**:

- PBTS doesn't override **routing** decision.  
Rather the use case is if routing have multiple TE tunnel paths toward destination,
then with PBTS we will be able to pick which TE tunnels should carry the traffic based on traffic's MPLS EXP or IP prec/DSCP values.

- **8 unique FC values** are supported (class 0 to class 7).  
Only one FC can be associated with a TE tunnel at any time.

- FC 0 is treated as the **default class** and doesn’t need to be explicitly configured under a TE tunnel.

- If a TE tunnel is not configured with an explicit FC, it will be associated with a
default FC 0.

- PBTS is supported on both **named** tunnels and **numbered** tunnels.  
However, using "named" tunnels is recommended.

- When many TE tunnels are equal best paths to same destination, they will be load balanced: up to 64 path **ECMP** TE tunnels can be used for a given destination.  
However, this can cause issue when we have more than 64 tunnels for a given PBTS destination.  
consider this config:  
  - tunnel-te1 up to tunnel-te64:  
  configured with FC3, use it for FC3 traffic to destination X.  
  - tunnel-te65 up to tunnel-te100:  
  configured with FC0, use it for FC0 traffic to destination X.  
  
If we have the above config, it could happen that only FC3 paths will be programmed, causing traffic blackhole for FC0 traffic.  
The recommendation is to have no more than 64 tunnel-te paths to a given destination, with 64 paths ECMP configured.

- **Fallback mechanism** is available:
  - When any FC (except FC 0) paths are down, traffic will switch to default class FC 0 path unless fallback PBTS class is configured.
  - If the fallback PBTS class path itself is not available, default class path will be used.
  - If both fallback PBTS class and default class paths are not available, then traffic will be dropped.

- All **TE related features** (such as TE-FRR etc.) will continue to work as is.


## Supported Software

This feature is introduced in **IOS XR 7.5.3**.

>
Note:
IOS XR 7.5.3 might not be a GA release, please check with your account team for feature availability.
{: .notice}


## Supported Hardware

This feature is supported on Cisco **8000** platform using **Gibraltar** ASIC.  
e.g.  
- fixed router: C8201-32FH  
- line cards for modular router: 88-LC0-36FH-M.


## Supported Scale

Up to **1,000 TE tunnels** per box and **64 paths ECMP** per destination.

Note:
With the following config enabled,

```
hw-module profile cef te-tunnel highscale-ldp-over-te-no-sr-over-srte
```

We can increase the scale of TE tunnels from 1,000 to 4,000.  
(applicable only when the router has all Gibraltar cards).


## Configuring PBTS : Basic Configuration

**Class-map** configuration.
```
        class-map match-any exp-0
           match mpls experimental topmost 0
        end-class-map
        !
        class-map match-any exp-1
           match mpls experimental topmost 1
        end-class-map
        !

        class-map match-any prec_5
            match precedence 5
        end-class-map
        !
        class-map match-any prec_6
            match precedence 6
        end-class-map
        !
        class-map match-any prec_7
            match precedence 7
        end-class-map
        !
        class-map match-any low_lat_af21
            match dscp af21
        end-class-map
        !
        class-map match-any bcast_vid_cs3
            match dscp cs3
        end-class-map
        !
        class-map match-any multimedia_conf_af41
            match dscp af41
        end-class-map
        !
```
String **class-map** with **forward-class** using **policy-map**.
```
        policy-map MY_PBTS
        !
        class exp-1
            set forward-class 1
        !
        class prec_5
            set forward-class 2
        !
        class prec_6
            set forward-class 3
        !
        class prec_7
            set forward-class 4
        !
        class low_lat_af21
            set forward-class 5
        !
        class bcast_vid_cs3
            set forward-class 6
        !
        class multimedia_conf_af41
            set forward-class 7
        !
        class class-default
        !
        end-policy-map
```

Assign the **policy-map** on **ingress interface** in inbound direction.
```
        interface Bundle-Ether1
            service-policy input MY_PBTS
        root
```

Assign **forward-class** to **TE tunnels**.
```
        interface tunnel-te0
         forward-class 0
        !
        interface tunnel-te1
         forward-class 1
        !
        interface tunnel-te2
         forward-class 2
        !
        interface tunnel-te3
         forward-class 3
        !
        mpls traffic-eng
         named-tunnels
          tunnel-te named_4
           forward-class 4
          !
          tunnel-te named_5
           forward-class 5
          !
          tunnel-te named_6
           forward-class 6
          !
          tunnel-te named_7
           forward-class 7
          !
         !
        !
```

When the above is configured, following PBTS path will be used:

`exp-0 traffic` ---> no explicit config within policy-map ; use default FC --> FC 0 --> `tunnel-te0`  
`exp-1 traffic` ---> explicit config within policy-map --> FC 1 --> `tunnel-te1`  
`prec_5 traffic` ---> explicit config within policy-map --> FC 2 --> `tunnel-te2`  
`prec_6 traffic` ---> explicit config within policy-map --> FC 3 --> `tunnel-te3`  
`prec_7 traffic` ---> explicit config within policy-map --> FC 4 --> `tunnel-te named_4`  
`low_lat_af21 traffic` ---> explicit config within policy-map --> FC 5 --> `tunnel-te named_5`  
`bcast_vid_cs3 traffic` ---> explicit config within policy-map --> FC 6 --> `tunnel-te named_6`  
`multimedia_conf_af41 traffic` ---> explicit config within policy-map --> FC 7 --> `tunnel-te named_7`  

`all other traffic` ---> no explicit config within policy-map ; use default FC --> FC 0 --> `tunnel-te0`  


## Configuring PBTS : Optional Configuration

**Optimizing hardware resource usage for forward-classes**

If we know for sure which FCs that we will use, we can optimize the hardware resource usage for those FCs by using the following config:
```
hw-module profile cef cbf forward-class-list <a list consisting of 0-7 values>
```
e.g.  
```
hw-module profile cef cbf forward-class-list 0 1 2 3 5
```
Note:
- Chassis needs to be reloaded for this config to go into effect.
- There won't be optimization taking place if all FC is listed.  
i.e.  
```
hw-module profile cef cbf forward-class-list 0 1 2 3 4 5 6 7
```
---

**Custom Fallback**

When TE tunnels associated with an FC go down, traffic can be redirected to another FC, any FC or chosen to be dropped via fallback PBTS configuration.  
This config will override the default behavior on the box when PBTS enabled TE tunnel goes down.
    
We can specify sequence of preferred fallback classes to revert to when TE tunnel of an FC goes down:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
    <span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH(config)#cef pbts class ?</span>
      <0-7>  Forward Class number
      any    Any forward class

    <span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH(config)#cef pbts class 7 ?</span>
      fallback-to  Fallback to configuration

    <span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH(config)#cef pbts class 7 fallback-to ?</span>
      <0-7>  Fallback Class number
      any    Fallback to any class
      drop   Fallback to drop
</code>
</pre>
</div>

For example, an operator that is using FC 0,1,2,3,5 can configure these following fallback paths:
```
    cef pbts class 0 fallback-to 1 2 3 5
    cef pbts class 1 fallback-to 0 2 3 5
    cef pbts class 2 fallback-to 0 1 3 5
    cef pbts class 3 fallback-to 0 1 2 5
    cef pbts class 5 fallback-to 0 1 2 3
```
In this way, first FC 0 and its paths will be present as fallback class for all other classes.  
If paths of FC 0 go down, then the next available FC in ascending order of 0-7 will be chosen as the fallback FC, i.e FC 1.  
When paths in FC 1 go down, the next FC 2 will be chosen as fallback so on and so forth.

We can also use `any` as fallback instead of specifying FC number.
In this way, next available FC in ascending order of 0-7 will be chosen as the fallback FC.
```
    cef pbts class 1 fallback-to any
```
or even:
```
    cef pbts class any fallback-to any
```

When “cef pbts class any fallback-to any” is specified, then the lowest available forward class ascending order of 0-7, is chosen as fallback for when paths to any class go down.
    
Keeping the above in mind, note that the following single line:
```
    cef pbts class any fallback-to any
```
will actually has same effect as configuring multiple config lines like these:
```
    cef pbts class 0 fallback-to 1 2 3 5
    cef pbts class 1 fallback-to 0 2 3 5
    cef pbts class 2 fallback-to 0 1 3 5
    cef pbts class 3 fallback-to 0 1 2 5
    cef pbts class 5 fallback-to 0 1 2 3
```


## Operation

### **Behavior when no PBTS is configured**

Let's start from the basic : router's default behavior when no PBTS is configured.

We're going to use following topology:

![02_topo.png]({{site.baseurl}}/images/02_topo.png)

Underlying network features:
- MPLS-TE cloud has ISIS for IGP, LDP for LDP-over-TE (LDPoTE), and MPLS TE tunnels.  
- 8 TE tunnels with router "Rean" as head end router (where we'll configure PBTS) and router "Musse" as tail end.  
- To allow for LDPoTE session to come up, 1 TE tunnel is also created on Musse as head end and Rean as tail end.  
- Configure these TE tunnels under LDP to bring up LDPoTE.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh mpls traffic-eng tunnels tabular</span>
Tue Oct 11 00:34:34.162 -07

           Tunnel   LSP     Destination          Source    Tun    FRR  LSP  Path
             Name    ID         Address         Address  State  State Role  Prot
----------------- ----- --------------- --------------- ------ ------ ---- -----
       tunnel-te0     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te1     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te2     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te3     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_4     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_5     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_6     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_7     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
</code>
</pre>
</div>
          
The LSP config:

```
group numbered_te
 interface 'tunnel-te.*'
  ipv4 unnumbered Loopback0
  autoroute announce
  !
  destination 202.158.0.2
  path-option 10 dynamic
 !
end-group


interface tunnel-te0
 apply-group numbered_te
 description exp_0
!
interface tunnel-te1
 apply-group numbered_te
 description exp_1
!
interface tunnel-te2
 apply-group numbered_te
 description exp_2
!
interface tunnel-te3
 apply-group numbered_te
 description exp_3
!

group named_te
 mpls traffic-eng
  named-tunnels
   tunnel-te '.*'
    path-option STATIC
     preference 10
     computation dynamic
    !
    autoroute announce
    !
    destination 202.158.0.2
   !
  !
 !
end-group

mpls traffic-eng
 named-tunnels
  tunnel-te named_4
   apply-group named_te
  !
  tunnel-te named_5
   apply-group named_te
  !
  tunnel-te named_6
   apply-group named_te
  !
  tunnel-te named_7
   apply-group named_te
  !
 !
!
```

Note that we have `autoroute announce` under all tunnels which means that these tunnels will be considered by IGP when forwarding traffic.
We also have `64 paths ECMP` configured to make sure all tunnels will be used.
```
router isis main
 address-family ipv4 unicast
  maximum-paths 64
 !
 address-family ipv6 unicast
  maximum-paths 64
 !
 ```
 
Here's CEF output for prefix "102.1.0.0/16" that is using the TE tunnels.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh cef 102.1.0.0/16</span>
Tue Oct 11 00:53:08.180 -07
102.1.0.0/16, version 2683, internal 0x1000001 0xf0 (ptr 0x940b61b0) [1], 0x400 (0x931456e8), 0x0 (0x0)
 Updated Oct 11 00:01:09.979
 Prefix Len 16, traffic index 0, precedence n/a, priority 15
  gateway array (0x92faa2d0) reference count 2, flags 0x0, source rib (7), 0 backups
                [3 type 3 flags 0x8401 (0xa96978c8) ext 0x0 (0x0)]
  LW-LDI[type=3, refc=1, ptr=0x931456e8, sh-ldi=0xa96978c8]
  gateway array update type-time 1 Oct 11 00:01:09.976
 LDI Update time Oct 11 00:32:31.661
 LW-LDI-TS Oct 11 00:32:31.665
   via 202.158.0.2/32, tunnel-te1, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 0 NHID 0x0 [0xa933e428 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, named_4, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 1 NHID 0x0 [0xa933eb18 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, named_5, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 2 NHID 0x0 [0xa933ed68 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, named_6, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 3 NHID 0x0 [0xa933efb8 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, named_7, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 4 NHID 0x0 [0xa933f208 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, tunnel-te0, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 5 NHID 0x0 [0xa933e1d8 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, tunnel-te2, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 6 NHID 0x0 [0xa933e678 0x0]
    next hop 202.158.0.2/32
    local adjacency
   via 202.158.0.2/32, tunnel-te3, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 7 NHID 0x0 [0xa933e8c8 0x0]
    next hop 202.158.0.2/32
    local adjacency

    Load distribution: 0 1 2 3 4 5 6 7 (refcount 3)

    Hash  OK  Interface                 Address
    0     Y   tunnel-te1                point2point    
    1     Y   named_4                   point2point    
    2     Y   named_5                   point2point    
    3     Y   named_6                   point2point    
    4     Y   named_7                   point2point    
    5     Y   tunnel-te0                point2point    
    6     Y   tunnel-te2                point2point    
    7     Y   tunnel-te3                point2point    
</code>
</pre>
</div>

Now let's try sending traffic flows toward destinations that fall within this subnet 102.1.0.0/16.
All flows have different /32 IP destination, but they all have same IP precedence field set with value 7.

"show interface accounting" output:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
named_4
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>9389</mark>          1032790

named_5
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>4224</mark>           464640

named_6
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>7981</mark>           877910

named_7
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>8918</mark>           980980

tunnel-te0
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>5633</mark>           619630

tunnel-te1
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>7981</mark>           877910

tunnel-te2
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>9854</mark>          1083940

tunnel-te3
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>6101</mark>           671110
</code>
</pre>
</div>

Note that:
- All tunnels are currently being used.  
Recall that by default all tunnels are assigned as FC0 where non-PBTS-classified traffic are forwarded.
- All tunnels are being utilized since we have ECMP configured.

This is behavior that we will see if PBTS is not configured.

### **Behavior when PBTS is configured for plain IP traffic.**

Now let's configure following PBTS config:
```
class-map match-any prec_7
 match precedence 7 
 end-class-map
! 
!
policy-map MY_PBTS
 class prec_7
  set forward-class 4
 ! 
 class class-default
 ! 
 end-policy-map
! 
interface BE1
 service-policy input MY_PBTS
!
mpls traffic-eng
 named-tunnels
  tunnel-te named_4
   forward-class 4
  !
 !
!
```

Note that with the above configured, we will have PBTS to match traffic with `IP prec 7` and send it to destination via tunnel-te `named_4`.

Commit that config and send traffic again.
Now you will see following result in "show interface accounting":

<div class="highlighter-rouge">
<pre class="highlight">
<code>
named_4
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0        <mark>87496</mark>          9624560

No accounting statistics available for named_5

No accounting statistics available for named_6

No accounting statistics available for named_7

No accounting statistics available for tunnel-te0

No accounting statistics available for tunnel-te1

No accounting statistics available for tunnel-te2

No accounting statistics available for tunnel-te3
</code>
</pre>
</div>

Now all traffic with IP prec 7 is sent via tunnel-te named_4 as configured.

Let's try sending other flows with `IP prec 5`, which TE tunnel it will take?  
Following result will show in "show interface accounting":

<div class="highlighter-rouge">
<pre class="highlight">
<code>
named_4
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0        58410          6425100

named_5
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>5941</mark>           653510

named_6
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>4591</mark>           505010

named_7
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>5941</mark>           653510

tunnel-te0
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>5672</mark>           623920

tunnel-te1
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>4864</mark>           535040

tunnel-te2
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>2973</mark>           327030

tunnel-te3
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>4593</mark>           505230
</code>
</pre>
</div>

We're sending 58,410 packets with IP prec 7, it's forwarded using TE tunnel named_4.
The rest of traffic is sent using other tunnels as we have ECMP.  
Recall that unclassified traffic will simply take TE tunnels that have no FC explicitly configured (i.e. tunnels that are assigned by default to forward class FC 0).

### **Behavior when PBTS is configured for MPLS traffic.**

We can also run LDP over these TE tunnels so that LDP FEC can also take benefits of PBTS.

Since LDP is using MPLS forwarding, we will deal with 2 types of incoming traffic for this use case:
- **MPLS encapsulated traffic**  
PBTS will classify based on MPLS EXP bits and then do label swap-push toward the matching TE tunnel.  
We don't classify MPLS traffic based on the inner payload.
- **Plain IP traffic**  
PBTS will classify based on IP prec/DSCP bits and then do label-push toward the matching TE tunnel.

Let's activate LDP now on router "Rean".
```
mpls ldp
 log
  neighbor
 !
 router-id 202.158.0.6
 interface Bundle-Ether1
 !
 interface tunnel-te0
 !
 interface tunnel-te1
 !
 interface tunnel-te2
 !
 interface tunnel-te3
 !
 interface tunnel-te named_4
 !
 interface tunnel-te named_5
 !
 interface tunnel-te named_6
 !
 interface tunnel-te named_7
 !
!
```

now sh cef shows `local label` and `labels imposed` (because LDP control plane is now in use).

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh cef 102.1.0.0/16</span>
Tue Oct 11 02:01:57.245 -07
102.1.0.0/16, version 2283, internal 0x1000001 0x30 (ptr 0x940b61b0) [1], 0x600 (0x93145a18), 0xa20 (0xa9875588)
 Updated Oct 11 02:00:00.630
 Prefix Len 16, traffic index 0, precedence n/a, priority 3, encap-id 0x1001000000001
  gateway array (0x92fa9d60) reference count 6, flags 0x68, source lsd (5), 1 backups
                [3 type 5 flags 0x208401 (0x94c754b8) ext 0x0 (0x0)]
  LW-LDI[type=5, refc=3, ptr=0x93145a18, sh-ldi=0x94c754b8]
  gateway array update type-time 1 Oct 11 01:59:56.590
 LDI Update time Oct 11 01:59:56.603
 LW-LDI-TS Oct 11 02:00:00.630
   via 202.158.0.2/32, tunnel-te1, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 0 NHID 0x0 [0xa933e300 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, named_4, 5 dependencies, weight 0, forward class 4 [flags 0x0]
    path-idx 1 NHID 0x0 [0xa933e9f0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, named_5, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 2 NHID 0x0 [0xa933ec40 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, named_6, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 3 NHID 0x0 [0xa933ee90 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, named_7, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 4 NHID 0x0 [0xa933f0e0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, tunnel-te0, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 5 NHID 0x0 [0xa933e0b0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, tunnel-te2, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 6 NHID 0x0 [0xa933e550 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>
   via 202.158.0.2/32, tunnel-te3, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 7 NHID 0x0 [0xa933e7a0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     <mark>local label 24014      labels imposed {ImplNull}</mark>

    Weight distribution:
    slot 0, weight 0, normalized_weight 1, class 0
    slot 1, weight 0, normalized_weight 1, class 0
    slot 2, weight 0, normalized_weight 1, class 0
    slot 3, weight 0, normalized_weight 1, class 0
    slot 4, weight 0, normalized_weight 1, class 0
    slot 5, weight 0, normalized_weight 1, class 0
    slot 6, weight 0, normalized_weight 1, class 0
    slot 7, weight 0, normalized_weight 1, forward class 4

    PBTS class information:
      class 0: 7 paths, offset 0
      forward class 4: 1 paths, offset 7
    Load distribution: 0 1 2 3 4 5 6 7 (refcount 3)

    Hash  OK  Interface                 Address
    0     Y   tunnel-te1                point2point    
    1     Y   named_5                   point2point    
    2     Y   named_6                   point2point    
    3     Y   named_7                   point2point    
    4     Y   tunnel-te0                point2point    
    5     Y   tunnel-te2                point2point    
    6     Y   tunnel-te3                point2point    
    7     Y   named_4                   point2point    
</code>
</pre>
</div>

Also pay attention to this part from output above that only shows when PBTS is active:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
    PBTS class information:
      class 0: 7 paths, offset 0                    <mark>🠔 7 TE tunnels for FC0 (hash 0 to hash 6)</mark>
      forward class 4: 1 paths, offset 7            <mark>🠔 1 TE tunnel for FC4 (hash 7)</mark>
    Load distribution: 0 1 2 3 4 5 6 7 (refcount 3)

    Hash  OK  Interface                 Address
    0     Y   tunnel-te1                point2point <mark>🠔 FC0 tunnel</mark>
    1     Y   named_5                   point2point <mark>🠔 FC0 tunnel</mark>
    2     Y   named_6                   point2point <mark>🠔 FC0 tunnel</mark>
    3     Y   named_7                   point2point <mark>🠔 FC0 tunnel</mark>
    4     Y   tunnel-te0                point2point <mark>🠔 FC0 tunnel</mark>
    5     Y   tunnel-te2                point2point <mark>🠔 FC0 tunnel</mark>
    6     Y   tunnel-te3                point2point <mark>🠔 FC0 tunnel</mark>
    7     Y   named_4                   point2point <mark>🠔 FC4 tunnel</mark>
</code>
</pre>
</div>
    
From sh cef output above, we also know that prefix 102.1.0.0/16 has local MPLS label 24014.  
We can also get PBTS information based on MPLS label info by referencing that local label.

This command will give same forwarding info as when we referenced the prefix instead of the label.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">sh cef mpls local-label 24014 eoS detail</span>
Tue Oct 11 02:16:23.976 -07
Label/EOS 24014/1, Label-type LDP, version 2283, internal 0x1000001 0x30 (ptr 0x94d4b178) [1], 0x600 (0x93145a18), 0xa20 (0xa9875588)
 Updated Oct 11 02:00:00.629
 Prefix Len 21, traffic index 0, precedence n/a, priority 3, encap-id 0x1001000000001
  gateway array (0x92fa9d60) reference count 6, flags 0x68, source lsd (5), 0 backups
                [3 type 5 flags 0x208401 (0x94c754b8) ext 0x0 (0x0)]
  LW-LDI[type=5, refc=3, ptr=0x93145a18, sh-ldi=0x94c754b8]
  gateway array update type-time 1 Oct 11 01:59:56.589
 LDI Update time Oct 11 01:59:56.602
 LW-LDI-TS Oct 11 02:00:00.629
   via 202.158.0.2/32, tunnel-te1, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 0 NHID 0x0 [0xa933e300 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_4, 5 dependencies, weight 0, forward class 4 [flags 0x0]
    path-idx 1 NHID 0x0 [0xa933e9f0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_5, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 2 NHID 0x0 [0xa933ec40 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_6, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 3 NHID 0x0 [0xa933ee90 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_7, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 4 NHID 0x0 [0xa933f0e0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te0, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 5 NHID 0x0 [0xa933e0b0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te2, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 6 NHID 0x0 [0xa933e550 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te3, 7 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 7 NHID 0x0 [0xa933e7a0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}

    Weight distribution:
    slot 0, weight 0, normalized_weight 1, class 0
    slot 1, weight 0, normalized_weight 1, class 0
    slot 2, weight 0, normalized_weight 1, class 0
    slot 3, weight 0, normalized_weight 1, class 0
    slot 4, weight 0, normalized_weight 1, class 0
    slot 5, weight 0, normalized_weight 1, class 0
    slot 6, weight 0, normalized_weight 1, class 0
    slot 7, weight 0, normalized_weight 1, forward class 4

    PBTS class information:
      class 0: 7 paths, offset 0
      forward class 4: 1 paths, offset 7
    Load distribution: 0 1 2 3 4 5 6 7 (refcount 3)

    Hash  OK  Interface                 Address
    0     Y   tunnel-te1                point2point    
    1     Y   named_5                   point2point    
    2     Y   named_6                   point2point    
    3     Y   named_7                   point2point    
    4     Y   tunnel-te0                point2point    
    5     Y   tunnel-te2                point2point    
    6     Y   tunnel-te3                point2point    
    7     Y   named_4                   point2point   
</code>
</pre>
</div>

Since now LDP control plane is active, let's try classify based on `MPLS EXP` in addition to the existing classification based on `IP prec`.  
We will configure PBTS to steer traffic with MPLS EXP 1 to tunnel-te1.
```
class-map match-any exp-1
 match mpls experimental topmost 1 
 end-class-map
! 
!
policy-map MY_PBTS
 class exp-1
  set forward-class 1
 ! 
 class prec_7
  set forward-class 4
 ! 
 class class-default
 ! 
 end-policy-map
! 
interface Bundle-Ether1
 service-policy input MY_PBTS
!
interface tunnel-te1
 apply-group numbered_te
 forward-class 1
!
```

Commit that and send traffic flow with MPLS EXP 1 and another flow of plain IP with prec 7.

Following result will show in "show interface accounting":

<div class="highlighter-rouge">
<pre class="highlight">
<code>
named_4
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0        <mark>29479</mark>          3242690

No accounting statistics available for named_5

No accounting statistics available for named_6

No accounting statistics available for named_7

No accounting statistics available for tunnel-te0

tunnel-te1
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0        <mark>11557</mark>          1225042

No accounting statistics available for tunnel-te2

No accounting statistics available for tunnel-te3
</code>
</pre>
</div>

We're sending 11,557 MPLS encapsulated packet with EXP bit 1, and it shows that they're forwarded using tunnel-te1.  
We're sending 29,479 plain IP packet with prec bit 7, and it shows that they're forwarded using tunnel named_4.


### **Behavior when default fallback is taking place.**

Now what happens if TE tunnel that is carrying PBTS steered traffic goes down?  
Let's see the fallback mechanism that is available by default.

Recall from previous example that we configured MPLS encapsulated packet with EXP bit 1 to be forwarded using tunnel-te1.

`exp-1 traffic` ➜ explicit config within policy-map ➜ FC 1 ➜ `tunnel-te1`

Also note that we have 8 TE tunnels, only 2 of which are configured with FC.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh run formal | i forward-class</span>
interface tunnel-te1 <mark>forward-class 1</mark>
mpls traffic-eng named-tunnels tunnel-te named_4 <mark>forward-class 4</mark>
</code>
</pre>
</div>

Now let's shut down tunnel-te1.
```
interface tunnel-te1
 shutdown
!
```

And send related traffic and show interface accounting:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>No accounting statistics available for named_4</mark>

named_5
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>2112</mark>           223872

named_6
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>2209</mark>           234154

named_7
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1826</mark>           193556

tunnel-te0
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>2113</mark>           223978

<mark>No accounting statistics available for tunnel-te1</mark>

tunnel-te2
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1825</mark>           193450

tunnel-te3
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>2210</mark>           234260
</code>
</pre>
</div>

Note that now exp-1 traffic is forwarded using all TE tunnels that are NOT configured with FC.  
These tunnels that have no FC configured are associated with FC0 and we fallback PBTS steered traffic by default to these tunnels when the primary tunnel goes down.

TE tunnel named_4, on the other hand, is not forwarding the fallback traffic since it's explicitly configured with FC4.

Now let's try to shut down all tunnels that have no FC configured.  
What happens with PBTS steered traffic if the fallback tunnels themselves are down?
```
interface tunnel-te0
 shutdown
!
interface tunnel-te2
 shutdown
!
interface tunnel-te3
 shutdown
!
mpls traffic-eng
 named-tunnels
  tunnel-te named_5
   shutdown
  !
  tunnel-te named_6
   shutdown
  !
  tunnel-te named_7
   shutdown
  !
 !
!
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh mpls traffic-eng tunnels tabular</span>
Wed Oct 12 02:28:14.189 -07

           Tunnel   LSP     Destination          Source    Tun    FRR  LSP  Path
             Name    ID         Address         Address  State  State Role  Prot
----------------- ----- --------------- --------------- ------ ------ ---- -----
       tunnel-te0     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
       tunnel-te1     0     202.158.0.2         0.0.0.0   down  Inact Head Inact
       tunnel-te2     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
       tunnel-te3     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
          named_4     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_5     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
          named_6     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
          named_7     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
</code>
</pre>
</div>

Send traffic again, and check interface accounting:
```
No accounting statistics available for named_4

No accounting statistics available for named_5

No accounting statistics available for named_6

No accounting statistics available for named_7

No accounting statistics available for tunnel-te0

No accounting statistics available for tunnel-te1

No accounting statistics available for tunnel-te2

No accounting statistics available for tunnel-te3
```

No TE tunnels are forwarding traffic now, traffic is actually being dropped as per the designed fallback behavior: 
  - When any FC (except FC 0) paths are down, traffic will switch to default class FC 0 path unless fallback PBTS class is configured.
  - If the fallback PBTS class path itself is not available, default class path will be used.
  - If both fallback PBTS class and default class paths are not available, then traffic will be dropped.

### **Behavior when custom fallback is taking place.**

We have discussed default fallback behavior above.  
We can actually customize the fallback behavior using following CLI config:
```
cef pbts class <> fallback-to <>
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
    <span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH(config)#cef pbts class ?</span>
      <0-7>  Forward Class number
      any    Any forward class

    <span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH(config)#cef pbts class 7 ?</span>
      fallback-to  Fallback to configuration

    <span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH(config)#cef pbts class 7 fallback-to ?</span>
      <0-7>  Fallback Class number
      any    Fallback to any class
      drop   Fallback to drop
</code>
</pre>
</div>

Let's try to customize fallback order in our setup.

First, bring up all TE tunnels and start fresh.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh mpls traffic-eng tunnels tabular</span> 
Wed Oct 12 02:36:01.791 -07

           Tunnel   LSP     Destination          Source    Tun    FRR  LSP  Path
             Name    ID         Address         Address  State  State Role  Prot
----------------- ----- --------------- --------------- ------ ------ ---- -----
       tunnel-te0     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te1     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te2     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te3     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_4     2     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_5     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_6     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_7     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
</code>
</pre>
</div>

Recall again that we have the following configured:

`exp-1 traffic` ➜ explicit config within policy-map ➜ FC 1 ➜ `tunnel-te1`

Also recall that we have 8 TE tunnels, only 2 of which are configured with FC.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh run formal | i forward-class</span>
interface tunnel-te1 <mark>forward-class 1</mark>
mpls traffic-eng named-tunnels tunnel-te named_4 <mark>forward-class 4</mark>
</code>
</pre>
</div>

Now let's configure the custom fallback as follows:
```
cef pbts class 1 fallback-to 4
```

Then shut tunnel-te1 again and send traffic.

You will get this output from show cef:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh cef mpls local-label 24014 eoS detail</span>
Wed Oct 12 02:46:58.227 -07
Label/EOS 24014/1, Label-type LDP, version 2340, internal 0x1000001 0x30 (ptr 0x94d4b178) [1], 0x600 (0x93133808), 0xa20 (0xa9ae8170)
 Updated Oct 12 02:46:46.581
 Prefix Len 21, traffic index 0, precedence n/a, priority 3, encap-id 0x1001000000001
  gateway array (0x92faa588) reference count 6, flags 0x68, source lsd (5), 0 backups
                [3 type 5 flags 0x208401 (0x94c6a2b0) ext 0x0 (0x0)]
  LW-LDI[type=5, refc=3, ptr=0x93133808, sh-ldi=0x94c6a2b0]
  gateway array update type-time 1 Oct 12 02:46:46.571
 LDI Update time Oct 12 02:46:46.577
 LW-LDI-TS Oct 12 02:46:46.581
   via 202.158.0.2/32, named_4, 5 dependencies, weight 0, forward class 4 [flags 0x0]
    path-idx 0 NHID 0x0 [0xa933e9f0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_5, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 1 NHID 0x0 [0xa933e300 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_6, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 2 NHID 0x0 [0xa933efb8 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_7, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 3 NHID 0x0 [0xa933e1d8 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te0, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 4 NHID 0x0 [0xa933e8c8 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te2, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 5 NHID 0x0 [0xa933e0b0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te3, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 6 NHID 0x0 [0xa933e7a0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}

    Weight distribution:
    slot 0, weight 0, normalized_weight 1, class 0
    slot 1, weight 0, normalized_weight 1, class 0
    slot 2, weight 0, normalized_weight 1, class 0
    slot 3, weight 0, normalized_weight 1, class 0
    slot 4, weight 0, normalized_weight 1, class 0
    slot 5, weight 0, normalized_weight 1, class 0
    slot 6, weight 0, normalized_weight 1, forward class 4

    PBTS class information:
      class 0: 6 paths, offset 0
      <mark>forward class 1: 1 paths, offset 6, fallback-to forward class 4</mark>
      forward class 4: 1 paths, offset 6
    Load distribution: 0 1 2 3 4 5 6 (refcount 3)

    Hash  OK  Interface                 Address
    0     Y   named_5                   point2point    
    1     Y   named_6                   point2point    
    2     Y   named_7                   point2point    
    3     Y   tunnel-te0                point2point    
    4     Y   tunnel-te2                point2point    
    5     Y   tunnel-te3                point2point    
    <mark>6     Y   named_4                   point2point</mark> 
</code>
</pre>
</div>

Note that since we shut tunnel-te1, now forward class 1 points to fallback tunnel named_4 as configured.

Traffic accounting result when we're sending FC 1 traffic:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
named_4
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>8683</mark>           920398

No accounting statistics available for named_5

No accounting statistics available for named_6

No accounting statistics available for named_7
          
No accounting statistics available for tunnel-te0

No accounting statistics available for tunnel-te1

No accounting statistics available for tunnel-te2

No accounting statistics available for tunnel-te3
</code>
</pre>
</div>

Now FC 1 traffic fell back to tunnel named_4 which is associated with FC4.  
Remember that without custom config, FC1 traffic will fall back to FC0 tunnels by default.

Similarly we can configure this kind of fallback for other FC.  
For instance:
```
cef pbts class 0 fallback-to 4
cef pbts class 1 fallback-to 4
cef pbts class 2 fallback-to 4
...
cef pbts class 7 fallback-to 4
```

or use this one line to cover all FCs:
```
cef pbts class any fallback-to 4
```

Now back to our example, what happens if the fallback FC4 TE tunnel also goes down?

Let's try that out, shut FC4 tunnel now:
```
mpls traffic-eng named-tunnels tunnel-te named_4 shut
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh mpls traffic-eng tunnels tabular</span>
Tue Oct 18 09:21:45.884 -07

           Tunnel   LSP     Destination          Source    Tun    FRR  LSP  Path
             Name    ID         Address         Address  State  State Role  Prot
----------------- ----- --------------- --------------- ------ ------ ---- -----
       tunnel-te0     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te1     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
       tunnel-te2     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
       tunnel-te3     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_4     0     202.158.0.2         0.0.0.0   <mark>down</mark>  Inact Head Inact
          named_5     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_6     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
          named_7     3     202.158.0.2     202.158.0.6     up  Inact Head Inact
</code>
</pre>
</div>


And send traffic again to FC1.

Here's the resulting traffic path for traffic with exp-1:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
No accounting statistics available for named_4

named_5
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1820</mark>           192920

named_6
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1906</mark>           202036

named_7
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1824</mark>           193344

tunnel-te0
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1734</mark>           183804

tunnel-te2
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>2169</mark>           229914

tunnel-te3
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>1651</mark>           175006
</code>
</pre>
</div>

Since tunnel-te1 is down, and the fallback path named_4 is also down, traffic will fall back again to FC0 TE tunnels.

We also have option to specify a list of FCs to be used as fallback.  
Instead of this:
```
cef pbts class 1 fallback-to 4
```

We can have something like this:
```
cef pbts class 1 fallback-to 4 5
```

With the above configured, let's now assign FC5 to tunnel-te named_5.
```
mpls traffic-eng named-tunnels tunnel-te named_5 forward-class 5
```

Since tunnel-te1 is down, fallback path named_4 is down, traffic should now use FC5 TE tunnel as per configuration, which is tunnel-te named_5.  
Let's see if that's really the path that the traffic would take:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<span style="background-color: #A0CFEC">RP/0/RP0/CPU0:Rean--C8201-32FH#sh cef mpls local-label 24014 eoS detail</span>
Sun Dec 11 23:45:45.887 -07
Label/EOS 24014/1, Label-type LDP, version 81, internal 0x1000001 0x30 (ptr 0x9a1e3550) [1], 0x600 (0x931b8378), 0xa20 (0xa9afb168)
 Updated Dec 11 23:26:14.199
 Prefix Len 21, traffic index 0, precedence n/a, priority 3, encap-id 0x1001300000001
  gateway array (0x92fea280) reference count 6, flags 0x68, source lsd (5), 0 backups
                [3 type 5 flags 0x200401 (0x9a1112b0) ext 0x0 (0x0)]
  LW-LDI[type=5, refc=3, ptr=0x931b8378, sh-ldi=0x9a1112b0]
  gateway array update type-time 4 Dec 11 23:43:48.828
 LDI Update time Dec 11 23:44:03.254
 LW-LDI-TS Dec 11 23:44:03.255
   via 202.158.0.2/32, named_5, 5 dependencies, weight 0, forward class 5 [flags 0x0]
    path-idx 0 NHID 0x0 [0xa97e1c40 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_6, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 1 NHID 0x0 [0xa97e1e90 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, named_7, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 2 NHID 0x0 [0xa97e20e0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te0, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 3 NHID 0x0 [0xa97e10b0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te2, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 4 NHID 0x0 [0xa97e1550 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}
   via 202.158.0.2/32, tunnel-te3, 5 dependencies, weight 0, class 0 [flags 0x0]
    path-idx 5 NHID 0x0 [0xa97e17a0 0x0]
    next hop 202.158.0.2/32
    local adjacency
     local label 24014      labels imposed {ImplNull}

    Weight distribution:
    slot 0, weight 0, normalized_weight 1, class 0
    slot 1, weight 0, normalized_weight 1, class 0
    slot 2, weight 0, normalized_weight 1, class 0
    slot 3, weight 0, normalized_weight 1, class 0
    slot 4, weight 0, normalized_weight 1, class 0
    slot 5, weight 0, normalized_weight 1, forward class 5

    PBTS class information:
      class 0: 5 paths, offset 0
      <mark>forward class 1: 1 paths, offset 5, fallback-to forward class 5</mark>
      forward class 5: 1 paths, offset 5
    Load distribution: 0 1 2 3 4 5 (refcount 3)

    Hash  OK  Interface                 Address
    0     Y   named_6                   point2point    
    1     Y   named_7                   point2point    
    2     Y   tunnel-te0                point2point    
    3     Y   tunnel-te2                point2point    
    4     Y   tunnel-te3                point2point    
    <mark>5     Y   named_5                   point2point</mark> 
</code>
</pre>
</div>

show interface accounting ...


<div class="highlighter-rouge">
<pre class="highlight">
<code>
No accounting statistics available for named_4

named_5
  Protocol              Pkts In         Chars In     Pkts Out        Chars Out
  MPLS                        0                0         <mark>4817</mark>           510602

No accounting statistics available for named_6

No accounting statistics available for named_7

No accounting statistics available for tunnel-te0

No accounting statistics available for tunnel-te2

No accounting statistics available for tunnel-te3
</code>
</pre>
</div>


Now traffic falls back to FC5 tunnel-te named_5 as expected.

That is all for basic PBTS operation overview.

Read on to get full configuration example for both head-end router and tail-end router.

## Full Configuration Example  

### Head-end / PBTS router

<div class="highlighter-rouge">
<pre class="highlight">
<code>
group named_te
 mpls traffic-eng
  named-tunnels
   tunnel-te '.*'
    path-option STATIC
     preference 10
     computation explicit chu_pyke_peyto_turin
    !
    autoroute announce
    !
    destination 202.158.0.2
    fast-reroute
   !
  !
 !
end-group
group numbered_te
 interface 'tunnel-te.*'
  ipv4 unnumbered Loopback0
  autoroute announce
  !
  destination 202.158.0.2
  fast-reroute
  path-option 10 explicit name chu_pyke_peyto_turin
 !
end-group
explicit-path name chu_pyke_peyto_turin
 index 1 next-address strict ipv4 unicast 103.1.3.3
 index 2 next-address strict ipv4 unicast 103.1.6.5
 index 3 next-address strict ipv4 unicast 103.1.7.2
!
ipv4 unnumbered mpls traffic-eng Loopback0
ipv4 source-route
interface Bundle-Ether1
 ipv4 address 103.1.1.6 255.255.255.0
 ipv6 address 103:1:1::6/64
 lacp mode on
!
interface Bundle-Ether2
 ipv4 address 103.1.3.6 255.255.255.0
 ipv6 address 103:1:3::6/64
 lacp mode on
!
interface Loopback0
 ipv4 address 202.158.0.6 255.255.255.255
 ipv6 address 202:158::6/128
!
interface tunnel-te0
 apply-group numbered_te
 forward-class 0
!
interface tunnel-te1
 apply-group numbered_te
 forward-class 1
!
interface tunnel-te2
 apply-group numbered_te
 forward-class 2
!
interface tunnel-te3
 apply-group numbered_te
 forward-class 3
!
router isis main
 net 47.0005.6666.6666.6666.00
 nsr
 distribute link-state
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  mpls traffic-eng level-1-2
  mpls traffic-eng router-id Loopback0
  router-id Loopback0
 !
 address-family ipv6 unicast
  metric-style wide
  router-id Loopback0
 !
 interface Bundle-Ether1
  point-to-point
  address-family ipv4 unicast
  !       
  address-family ipv6 unicast
  !
 !
 interface Bundle-Ether2
  point-to-point
  address-family ipv4 unicast
  !
  address-family ipv6 unicast
  !
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
  !
  address-family ipv6 unicast
  !
 !
!
rsvp
 interface Bundle-Ether2
  bandwidth 10000000
 !
!
mpls traffic-eng
 interface Bundle-Ether2
 !
 logging events all
 auto-tunnel backup
  timers removal unused 5
  tunnel-id min 65000 max 65500
 !
 reoptimize 604800
 bandwidth-accounting
 !
 soft-preemption
 !
 named-tunnels
  tunnel-te named_4
   apply-group named_te
   forward-class 4
  !
  tunnel-te named_5
   apply-group named_te
   forward-class 5
  !
  tunnel-te named_6
   apply-group named_te
   forward-class 6
  !
  tunnel-te named_7
   apply-group named_te
   forward-class 7
  !
 !
!
mpls ldp
 log
  neighbor
 !
 router-id 202.158.0.6
 interface Bundle-Ether1
 !
 interface tunnel-te0
 !
 interface tunnel-te1
 !
 interface tunnel-te2
 !
 interface tunnel-te3
 !
 interface tunnel-te named_4
 !
 interface tunnel-te named_5
 !
 interface tunnel-te named_6
 !
 interface tunnel-te named_7
 !
!

<mark>PBTS configuration</mark>

class-map match-any exp-0
   match mpls experimental topmost 0
end-class-map
!
class-map match-any exp-1
   match mpls experimental topmost 1
end-class-map
!

class-map match-any prec_5
match precedence 5
end-class-map
!
class-map match-any prec_6
match precedence 6
end-class-map
!
class-map match-any prec_7
match precedence 7
end-class-map
!
class-map match-any low_lat_af21
match dscp af21
end-class-map
!
class-map match-any bcast_vid_cs3
match dscp cs3
end-class-map
!
class-map match-any multimedia_conf_af41
match dscp af41
end-class-map
!

policy-map MY_PBTS
!
class exp-1
set forward-class 1
!
class prec_5
set forward-class 2
!
class prec_6
set forward-class 3
!
class prec_7
set forward-class 4
!
class low_lat_af21
set forward-class 5
!
class bcast_vid_cs3
set forward-class 6
!
class multimedia_conf_af41
set forward-class 7
!
class class-default
!
end-policy-map


interface Bundle-Ether1
service-policy input MY_PBTS
root
</code>
</pre>
</div>

### Tail-end router

No special config is needed here, as PBTS is only configured on head-end router.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
explicit-path name to_rean
 index 1 next-address strict ipv4 unicast 200.4.0.1
 index 2 next-address strict ipv4 unicast 103.1.3.6
!
ipv4 unnumbered mpls traffic-eng Loopback0
interface Loopback0
 ipv4 address 202.158.0.2 255.255.255.255
 ipv6 address 202:158::2/128
!
interface tunnel-te1
 ipv4 unnumbered Loopback0
 autoroute announce
 !
 destination 202.158.0.6
 path-option 10 explicit name to_rean
!
interface TenGigE0/0/0/0.1
 ipv4 address 200.4.0.2 255.255.0.0
 ipv6 address 200:4::2/64
 encapsulation dot1q 1
!
interface TenGigE0/0/0/32.1
 ipv4 address 102.1.1.1 255.255.0.0
 ipv6 address 102:1:1::1/64
 encapsulation dot1q 1
!
router isis main
 net 47.0005.2222.2222.2222.00
 nsr
 distribute link-state
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  mpls traffic-eng level-1-2
  mpls traffic-eng router-id Loopback0
  router-id Loopback0
 !
 address-family ipv6 unicast
  metric-style wide
  router-id Loopback0
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
  !
  address-family ipv6 unicast
  !
 !
 interface TenGigE0/0/0/0.1
  point-to-point
  address-family ipv4 unicast
  !
  address-family ipv6 unicast
  !
 !
 interface TenGigE0/0/0/32.1
  passive
  address-family ipv4 unicast
  !
  address-family ipv6 unicast
  !
 !
!
rsvp
 interface TenGigE0/0/0/0.1
  bandwidth 10000000
 !
!
mpls traffic-eng
 interface TenGigE0/0/0/0.1
 logging events all
 reoptimize 604800
 bandwidth-accounting
 !
 soft-preemption
 !
!
mpls ldp
 router-id 202.158.0.2
 interface tunnel-te1
 !
!
</code>
</pre>
</div>

## Verification Commands


See PBTS path based on IP info:
```
	sh cef 102.1.0.0/16
```

See PBTS path based on MPLS LDP info:
```
	sh mpls ldp forwarding 101.1.0.0/16 (grab `Label In`)
	sh cef mpls local-label <label_in> eoS detail
```

Check if PBTS policy map is in use:
```
	sh qos interface <ingress interface> input
```
	
Check whether traffic is taking correct tunnel-te:
```
	sh interface tunnel-te <> accounting
```


## Logs to get in case of issues



<div class="highlighter-rouge">
<pre class="highlight">
<code>

Basic info:

<span style="background-color: #A0CFEC">show ofa trace</span> | inc forward_class | file harddisk:/0614a_show_ofa_trace.txt
<span style="background-color: #A0CFEC">show cef</span> 102.1.0.0/16 hardware egress loc 0/RP0/CPU0 | file harddisk:/0614a_show_cef_hw_egress_ipv4.txt
<span style="background-color: #A0CFEC">show cef ipv6</span> 102:1:1::2 hardware egress loc 0/RP0/CPU0 | file harddisk:/0614a_show_cef_hw_egress_ipv6.txt
<span style="background-color: #A0CFEC">show tech platform-fwd</span> file harddisk:/0614a_show_tech_platform_fwd
<span style="background-color: #A0CFEC">show tech ofa</span> file harddisk:/0614a_show_tech_ofa
<span style="background-color: #A0CFEC">show tech fabric</span> file harddisk:/0614a_show_tech_fabric
<span style="background-color: #A0CFEC">show tech mpls lsd</span> file harddisk:/0614a_show_tech_mpls_lsd
<span style="background-color: #A0CFEC">show tech mpls traffic-eng</span> file harddisk:/0614a_show_tech_mpls_te
<span style="background-color: #A0CFEC">show tech cef</span> file harddisk:/0614a_show_tech_cef

SDK chain info:

<span style="background-color: #A0CFEC">show controllers npu debugshell 0</span> "script print_prefix 0 102.1.0.0" loc 0/RP0/CPU0 | file harddisk:/0614a_show_sdk_chain_ipv4.txt
<span style="background-color: #A0CFEC">show controllers npu debugshell 0</span> "script print_prefix 0 102:1:1::" loc 0/RP0/CPU0 | file harddisk:/0614a_show_sdk_chain_ipv6.txt
</code>
</pre>
</div>


## Glossary

**EXP** : "Experimental" bits field on an MPLS header  
**Prec** : "Precedence" bits field on an IPv4 header  
**DSCP** : "Differentiated Service Code Point" bits field on an IPv4/IPv6 header  
**TE** : Traffic engineering  
**TE tunnel / tunnel-te** : MPLS TE tunnel created by user to carry traffic via label switching  
**FC** : Forward-class  
**LDPoTE** : LDP-over-TE  
**ECMP** : Equal cost multi paths
