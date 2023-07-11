---
published: true
date: '2023-07-11 11:46 +0200'
title: 'Cisco 8800 Fan Tray, Fabric Card & Line Card Replacement'
excerpt: >-
  This blog post details the steps required to replace a FT, FC and LC on a
  Cisco 8800 router. 
position: hidden
tags:
  - Hardware
  - Method of Procedure
  - Cisco 8000
author: Manish Chandra Pandey
---
{% include toc title="Cisco 8800 Fan Tray, Fabric Card & Line Card Replacement" %}

## Introduction

On a Cisco 8800 router, Fan Tray (FT), Fabric Cards (FC) and Line Cards (LC) are Field Replaceable Units (FRU).  
8800 distributed systems use four Fan Trays but can operate with three fan trays while operator replaces one, or temporarily remove one to replace one of the Fabric Cards sitting behind the Fan Tray.  When a Fan Tray is removed, the remaining Fan Trays speed up their fans to maintain the designed airflow.  
Fabric Cards sit behind the Fan Trays on the chassis and the operator must remove Fan Tray unit to replace a Fabric card. 
This blog post details the steps required to replace a FT, FC and LC on a Cisco 8800 router. A video with detailed steps for FT and FC replacement is also linked in this post below and takes  8818 chassis as an example. At the time of publishing this video, the fabric card rail extension is only available on 8818 routers. All other steps are applicable to other chassis (8804, 8808, 8812).

## Before Getting Started

Verify the following –
- Line card inserted
<code>show platform</code>
- IOS XR version
<code>show install active summary</code>
<code>show install committed summary</code>
- FPD status of the cards
<code>show hw-module fpd</code> (should be “CURRENT”)


## Fan Tray/Fabric Card Replacement

Fabric cards mount vertically into the chassis connecting orthogonally to the horizontally placed line cards in the system. Fabric cards utilize ejectors on them to gracefully handle card insertion and removal from the chassis. Ejector handles should be operated properly as directed in this article/video for smooth and uninterrupted functioning of the card in the system.

### Video

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=sQtI-uAkUGQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Step-by-step Procedure

Summary of steps required to replace a FC are as below – 

1. Shutdown fabric plane.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8800(config)#controller fabric plane 0 shutdown    
RP/0/RP0/CPU0:8800(config)#commit

RP/0/RP0/CPU0:May 23 22:39:28.824 UTC: fsdbagg[225]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-0 :DECLARE :: Fabric Plane-0 DOWN 
RP/0/RP0/CPU0:May 23 22:39:28.824 UTC: fsdbagg[225]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [5117] : Plane 0 state changed to DOWN 

RP/0/RP0/CPU0:8800#show controllers fabric plane all 

Plane Admin Plane    up->dn  up->mcast
Id    State State    counter   counter
--------------------------------------
<mark>0     DN    DN             1         0 </mark>
1     UP    UP             0         0 
2     UP    UP             0         0 
3     UP    UP             0         0 
4     UP    UP             0         0 
5     UP    UP             0         0 
6     UP    UP             0         0 
7     UP    UP             0         0 
RP/0/RP0/CPU0:8800#
</code>
</pre>
</div>
2. Shutdown fabric card associated to fabric plane.
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8800(config)#hw-module shutdown location 0/FC0
RP/0/RP0/CPU0:8800(config)#commit
RP/0/RP0/CPU0:May 23 22:42:13.698 UTC: shelfmgr[240]: %PLATFORM-SHELFMGR-4-CARD_SHUTDOWN : Shutting down 0/FC0: User initiated shutdown from config 
RP/0/RP0/CPU0:May 23 22:42:13.698 UTC: shelfmgr[240]: %PLATFORM-SHELFMGR-6-INFO_LOG : 0/FC0 is shutdown 
RP/0/RP0/CPU0:8800#show platform 
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        8800-RP(Active)          IOS XR RUN               NSHUT
0/RP0/BMC0        8800-RP                  OPERATIONAL              NSHUT
0/0/CPU0          8800-LC-48H              IOS XR RUN               NSHUT
<mark>0/FC0             8808-FC                  SHUT DOWN                SHUT</mark>
0/FC1             8808-FC                  OPERATIONAL              NSHUT
</code>
</pre>
</div>
3. Verify Fabric Card LED is turned Off.  
![8800-fabric-led.png]({{site.baseurl}}/images/8800-fabric-led.png)
4.	Unscrew FAN tray  
![fabric-unscrew.png]({{site.baseurl}}/images/fabric-unscrew.png)
5.	Remove Fan Tray corresponding to the fabric card which needs to be replaced.  
![ft-removal.png]({{site.baseurl}}/images/ft-removal.png)
6.	Insert Fabric card rail extension. (note: specific to 8818 chassis)  
![fc-rail.png]({{site.baseurl}}/images/fc-rail.png)
7.	Loosen both the ejector handles on fabric card.  
![fc-ejector-1.png]({{site.baseurl}}/images/fc-ejector-1.png)
8.	Pull the FC out of the chassis using both ejector handles.  
![fc-ejector-2.png]({{site.baseurl}}/images/fc-ejector-2.png)
9.	Insert replacement fabric card.  
![fc-insertion.png]({{site.baseurl}}/images/fc-insertion.png)
10.	Tighten both the ejector handles on the new fabric card.
11.	Remove fabric card rail extension.
12.	Install Fan tray.
13.	Unshut fabric card.
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8800(config)#no hw-module shutdown location 0/FC0
RP/0/RP0/CPU0:8800(config)#commit
RP/0/RP0/CPU0:May 23 22:45:02.113 UTC: shelfmgr[240]: %PLATFORM-SHELFMGR-4-CARD_RELOAD : Reloading 0/FC0: User initiated no-shutdown from config 
RP/0/RP0/CPU0:ios#RP/0/RP0/CPU0:May 23 22:45:34.435 UTC: shelfmgr[240]: %PLATFORM-SHELFMGR-6-INFO_LOG : 0/FC0 is operational 
RP/0/RP0/CPU0:ios#show platform
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        8800-RP(Active)          IOS XR RUN               NSHUT
0/RP0/BMC0        8800-RP                  OPERATIONAL              NSHUT
0/0/CPU0          8800-LC-48H              IOS XR RUN               NSHUT
<mark>0/FC0             8808-FC                  OPERATIONAL              NSHUT</mark>
0/FC1             8808-FC                  OPERATIONAL              NSHUT
</code>
</pre>
</div>
14.	Unshut fabric plane. 
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8800(config)#no controller fabric plane 0 shutdown 
RP/0/RP0/CPU0:8800(config)#commit
RP/0/RP0/CPU0:May 23 22:47:09.904 UTC: fsdbagg[225]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [5117] : Plane 0 state changed to UP 
RP/0/RP0/CPU0:ios#show controllers fabric plane all 
Plane Admin Plane    up->dn  up->mcast
Id    State State    counter   counter
--------------------------------------
<mark>0     UP    UP             1         0</mark>
1     UP    UP             0         0 
2     UP    UP             0         0 
3     UP    UP             0         0 
4     UP    UP             0         0 
5     UP    UP             0         0 
6     UP    UP             0         0 
7     UP    UP             0         0 
RP/0/RP0/CPU0:8800#
</code>
</pre>
</div>

**Note:** Too ensure proper airflow and prevent overheating, do not operate the router with three fan trays for more than 10 minutes. 8800 routers can work with 3 FTs and operates in N+1 redundancy model. If the replacement fan tray is not ready, faulty FT must remain in the chassis.
{: .notice}

## Line Card Replacement

Line cards on 8800 chassis are available in two variants with respect to the ejector functionality. 

- LC Type1: Ejector Levers with Buttons
- LC Type2: Ejector Levers with Latches

Line cards have two ejector levers to release the card from the router. Use these levers to remove the line card and to seat the line card firmly in the router when line card is installed. The ejector levers align and seat the card connectors in the router. To avoid damaging card mechanical components, never carry a line card by the captive installation screws or ejector levers. Doing so can damage these components and cause card insertion problems.  

Opening the ejector levers of an installed line card causes the line card to shut down even when the captive screws are screwed in. To reboot the line card, users must do one of the following:

- Remove and reinsert the line card and close the ejector levers.
- Close the ejector levers and reload the card using the <code>reload location 0/line-card-slot force</code> command.

### Video

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=t9Ly9re5g3s" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Line Card Removal

1. Run the <code>shutdown location 0/linecard-slot/CPU0</code> command, which gracefully shuts down the line card.
2. Verify that the Line Card LED for the slot that you specified turns off. Also, you can use the show platform command to verify that the status of the card is SHUT DOWN.
3.	Loosen the two captive screws.
4.	Ejectors
  - LC Type1:  Press the ejector buttons to open two ejector levers.
  - LC Type2: Slide the ejector lever latches outward and pull the two ejector levers. 
5.	Use the ejector levers to pull the line card a couple of inches (about 5 cm) from the chassis.
6.	Close the ejector levers. Use one hand to hold the front of the line card, place your other hand under the line card to support its weight, pull it out of the chassis, and set it on an antistatic surface or inside an antistatic bag.

### Line Card Insertion
  
1.	Align the back of the line card to the guides in the open line card slot and slide the line card all the way into the slot.
2.	The line card stops when its front is about 0.25 inches (0.6 cm) outside the front of the chassis. 
3. Ejectors
  - LC Type1: Press the ejector buttons, and the two levers move partway from the center of the line card.
  - LC Type2: Slide the ejector lever latches to unlock the ejector levers and open the ejector levers outward.
4.	Close the ejectors by rotating the ends of the two levers toward the center of the chassis. 
5.	Tighten the two captive screws. (Fully tighten both captive installation screws to ensure that the card is correctly seated in the router. A card that is only partially seated in the router might not operate properly, even if it boots.)
6.	Attach each interface cable to the appropriate port on the line card. Use the label on each cable to determine which port each cable attaches to. 
7.	Verify that the line card LEDs turn on and appear green.

## General Verification

<div class="highlighter-rouge">
<pre class="highlight">
<code>
show shelfmgr history oir
show inventory
show proc cpu
show environment all
show led
show alarms
show controller fabric health
show controllers fabric plane all statistics
show hw-module fpd
</code>
</pre>
</div>


## Conclusion

This concludes the steps required for replacement of FT, FC and LC on 8800 system. Following meticulously to the outlined steps provided in this document will result in smooth operations.
