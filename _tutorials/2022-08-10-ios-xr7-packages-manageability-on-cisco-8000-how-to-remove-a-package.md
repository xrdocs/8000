---
published: true
date: '2022-08-10 11:47 +0200'
title: 'IOS XR7 packages manageability on Cisco 8000: how to remove a package'
position: hidden
author: Frederic Cuiller
excerpt: >-
  This blog post aims to describe IOS XR7 software operation details to remove
  optional packages. 
tags:
  - iosxr
  - XR7
  - Cisco 8000
---
## Introduction 

This blog post aims to describe IOS XR7 software operation details to remove optional packages.

## History 

Cisco IOS-XR is a modular Network Operating System created in 2004 for CRS. It was originally built using a 32-bit QNX based micro kernel architecture. It eventually evolved toward a 64-bit Linux-based kernel with IOS-XR 6. Those 2 versions are referenced as classic IOS-XR (cXR) and IOS-XR 64bit (or enhanced IOS-XR or eXR).  

Both of those IOS-XR flavors rely on packages concept. There are 3 types of packages: 
1. Cisco IOS XR Unicast Routing Core Bundle (also referred as mini.pie or mini.iso). it contains: 
- Operating system (OS) and minimum boot image (MBI)—Kernel, file system, memory management, and other slow changing core components.  
- Base—Interface manager, system database, checkpoint services, configuration management, other slow-changing components.  
- Infra—Resource management: rack, fabric. 
- Routing - RIB, BGP, ISIS, OSPF, EIGRP, RIP, RPL, and other routing protocols. 
- Forwarding - FIB, ARP, QoS, ACL, and other components. 
- LC — Line card drivers.  

2. Individually-Installable Optional Package. It contains software for specific features such multicast, MPLS or manageability. 

3. Software Maintenance Upgrades (SMU). It contains fixes for a specific defect.

## Cisco 8000 and IOS XR7 

Cisco 8000 runs IOS XR7 software. IOS XR7 is built on top of the WindRiver Linux 9 distribution. One major change is the complete removal of the admin plane. In addition, IOS XR control plane processes now run natively on the host.  

Cisco IOS XR7 is composed of a base image (ISO) that provides the XR infrastructure. The ISO image is made up of a set of packages (also called RPMs). These packages are of two types:  

- A mandatory package that is included in the ISO

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active <mark>all</mark>
Tue Aug 24 17:03:27.152 UTC 
Package                                                                  Version 
---------------------------------------------------- --------------------------- 
<mark>8000-boot-scripts                                                7.3.15v1.0.0-r0</mark>
8000-cpa-setup-x86                                               7.3.15v1.0.0-r0 
8101-32h-cpa-sb-x86                                              7.3.15v1.0.0-r0 
8102-64h-cpa-sb-x86                                              7.3.15v1.0.0-r0 
8201-32fh-cpa-sb-x86                                             7.3.15v1.0.0-r0 
8201-cpa-sb-x86                                                  7.3.15v1.0.0-r0 
8202-cpa-sb-x86                                                  7.3.15v1.0.0-r0 
8804-fc-data-cpa-sb-x86                                          7.3.15v1.0.0-r0 
acl                                                            2.2.52-r0.23.38.0 
snip
</code>
</pre>
</div>
  
 - An optional package that is included in the ISO

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active ? 
all      Show all user-installable packages 
<mark>summary  Show a summary of active packages - optional and bugfix packages only</mark> 

RP/0/RP0/CPU0:8201-32FH-1#sh install active summary 
Tue Aug 24 12:19:30.076 UTC 
Active Packages:    XR: 178    All: 1274 
Label:              7.3.15 
Software Hash:      9149973e08793c3f44e84a4a5b385c8a 
<mark>Optional Packages                                                        Version</mark>
---------------------------------------------------- --------------------------- 
xr-8000-mcast                                                     7.3.15v1.0.0-1 
xr-8000-netflow                                                   7.3.15v1.0.0-1 
xr-bgp                                                            7.3.15v1.0.0-1 
xr-ipsla                                                          7.3.15v1.0.0-1 
xr-is-is                                                          7.3.15v1.0.0-1 
xr-lldp                                                           7.3.15v1.0.0-1 
xr-mcast                                                          7.3.15v1.0.0-1 
xr-mpls-oam                                                       7.3.15v1.0.0-1 
xr-netflow                                                        7.3.15v1.0.0-1 
xr-ospf                                                           7.3.15v1.0.0-1 
xr-perfmgmt                                                       7.3.15v1.0.0-1 
xr-track                                                          7.3.15v1.0.0-1 
</code>
</pre>
</div>
 
Last type of package is optional package that is not included in the ISO. They can be downloaded on cisco.com website. 

![XR7-CCO.png]({{site.baseurl}}/images/XR7-CCO.png)

  
