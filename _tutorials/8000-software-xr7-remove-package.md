---
published: true
date: '2022-08-10 11:47 +0200'
title: 'Cisco 8000 software manageability: how to remove a package'
position: top
author: Frederic Cuiller
excerpt: >-
  This blog post aims to describe IOS XR7 software operation details to remove
  optional packages. 
tags:
  - iosxr
  - XR7
  - Cisco 8000
---
{% include toc icon="table" title="Cisco 8000 software manageability: how to remove a package" %}

## Introduction 

This blog post aims to describe IOS XR7 software operation details to remove optional packages from a Cisco 8000 router.

## History 

Cisco IOS-XR is a modular Network Operating System created in 2004 for CRS. It was originally built using a 32-bit QNX based micro kernel architecture. It then evolved toward a 64-bit Linux-based kernel with IOS-XR 6. Those 2 versions are referenced as classic IOS-XR (cXR) and IOS-XR 64bit (or enhanced IOS-XR or eXR).  

Both of those IOS-XR flavors rely on packages concept. There are 3 types of packages: 
1. Cisco IOS XR Unicast Routing Core Bundle (also referred as mini.pie or mini.iso). It contains: 
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

![XR7-architecture.png]({{site.baseurl}}/images/XR7-architecture.png)

**Info:** IOS XR7 (also referred as Lindt) is a XR software architecture evolution. It's currently applicable to Cisco 8000, NCS 540 and NCS-57B1 series. A software release can contain the number 7 but still follows cXR or eXR software architecture (e.g ASR 9000 XR 7.5.2 release is **not** XR7 architecture) 
See datasheet [here](https://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/ios-xr-software/datasheet-c78-743014.html) for more information.
{: .notice--primary}

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
 
Last type of package is optional package that is **not** included in the ISO. They can be downloaded on cisco.com website. 

![XR7-CCO.png]({{site.baseurl}}/images/XR7-CCO.png)

For IOS-XR 7.3.15, those optional packages are:
- cdp
- telnet
- healthcheck
- k9 (crypto dataplane)

**Info:** telnet is now optional because ssh is part of the default image. Starting IOS XR 7.0.1, k9sec package is no more required for ssh/sftp features.  This is applicable for XR 32bits, XR 64bits and XR7. Fore more information, please check "SSH and SFTP in Baseline Cisco IOS XR Software Image" section in [**System Security Configuration Guide**](https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/security/70x/b-system-security-cg-cisco8000-70x/implementing-secure-shell.html#concept_url_rxk_m3b)  
k9 package is still required for dataplane features such MACsec.
{: .notice--primary}

## Removing optional packages included in base image
Following a USB boot, operator might want to disable some optional packages to limit attack surface or optimize memory footprint.  
For instance, while multicast is not enabled in this example, mrib process is spawned and its associated threads are in Sleeping state:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh run multicast-routing
Tue Aug 24 19:18:54.009 UTC
<mark>% No such configuration item(s)</mark>

RP/0/RP0/CPU0:8201-32FH-1#sh processes mrib
Tue Aug 24 19:17:52.150 UTC
                  Job Id: 1151
                     PID: 6090
            Process name: mrib
         Executable path: /opt/cisco/install-iosxr/base/bin/mrib
              Instance #: 1
              Version ID: 00.00.0000
                 Respawn: ON
           Respawn count: 1
            Last started: Thu Aug 19 21:38:42 2021
           <mark>Process state: Run</mark>
           Package state: Normal
           Process group: mcast-routing
                    core: MAINMEM
               Max. core: 0
                   Level: 170
               Placement: Placeable
            startup_path: /opt/cisco/install-iosxr/base/startup/mrib.startup
                   Ready: 87.313s
        Process cpu time: 73.510 user, 44.100 kernel, 117.610 total
JID    TID  Stack  pri  <mark>state</mark>        NAME             rt_pri
1151   6090    0K  20   <mark>Sleeping</mark>     mrib             0
1151   6170    0K  20   Sleeping     lwm_service_thr  0
1151   6176    0K  20   Sleeping     qsm_service_thr  0
1151   6236    0K  20   Sleeping     mrib             0
1151   6242    0K  20   Sleeping     evm_signal_thre  0
1151   6514    0K  20   Sleeping     mrib             0
1151   6516    0K  20   Sleeping     aipc-lrd_thread  0
1151   6517    0K  20   Sleeping     chkpt_evm        0
1151   6536    0K  20   Sleeping     mrib             0
1151   6538    0K  20   Sleeping     mrib             0
1151   6588    0K  20   Sleeping     sf_mrib_chkpt    0
1151   6589    0K  20   Sleeping     mrib             0
1151   7399    0K  20   Sleeping     mrib_infra_thre  0
1151   7402    0K  20   Sleeping     amt-stats        0
1151   7404    0K  20   Sleeping     mrib             0
1151   7447    0K  20   Sleeping     mrib_infra_thre  0
</code>
</pre>
</div>

Even if insignificant, this process also consumes memory:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh processes memory 6090
Tue Aug 24 19:18:13.993 UTC
JID      Text(KB)   Data(KB)  Stack(KB) Dynamic(KB) Process
------ ---------- ---------- ---------- ----------- ------------------------------
1151         1516      26352        132        7996 mrib
</code>
</pre>
</div>

In order to get rid of unused processes, an operator could decide to remove an optional package. xr-mcast will be used in the next example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active summary
Tue Aug 24 19:41:42.810 UTC
Active Packages:    XR: 178    All: 1274
Label:              7.3.15
Software Hash:      9149973e08793c3f44e84a4a5b385c8a

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
<mark>xr-8000-mcast                                                     7.3.15v1.0.0-1</mark>
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

The first step is to remove the package with *install package remove* command:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#install package ?
  abort      Abort the specified packaging operation(s)
  add        Add packages
  downgrade  Downgrade packages
  <mark>remove     Remove packages</mark>
  rollback   Rollback to the software committed by the given transaction id
  upgrade    Upgrade packages
  replace    Replace currently installed software with that in a given ISO

RP/0/RP0/CPU0:8201-32FH-1#install package remove xr-8000-mcast synchronous
Tue Aug 24 19:42:11.147 UTC
Starting:
  install package remove xr-8000-mcast
Packaging operation 1.1.1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing ...
Current activity: Veto check
Current activity: Package add or other package operation .

Packaging operation 1.1.1: 'install package remove xr-8000-mcast' completed without error
RP/0/RP0/CPU0:8201-32FH-1#
</code>
</pre>
</div>

After this step, package is still active:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active summary
Tue Aug 24 19:43:40.569 UTC
Active Packages:    XR: 178    All: 1274
Label:              7.3.15
Software Hash:      9149973e08793c3f44e84a4a5b385c8a

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
<mark>xr-8000-mcast                                                     7.3.15v1.0.0-1</mark>
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

Note it’s still available in the repository:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install available
Tue Aug 24 19:44:22.618 UTC
Trying to access repositories...

Package                                              Architecture                         Version Repository                          Cached
---------------------------------------------------- ---------------- --------------------------- ----------------------------------- ------
xr-8000-mcast                                        x86_64                        7.3.15v1.0.0-1                                     Y
xr-mcast                                             x86_64                        7.3.15v1.0.0-1                                     Y
snip
</code>
</pre>
</div>

Second step is to apply the change. This is accomplished with *install apply* command:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#install ?
  <mark>apply     Apply the latest atomic change</mark>
  attach    Attach to a running install operation
  commit    Commit the transaction
  package   Package operations
  rollback  Rollback to the software committed by the given transaction id and apply the change
  source    Install or upgrade packages from the specified source and apply the change
  replace   Replace currently installed software with that in a given ISO and apply the change

RP/0/RP0/CPU0:8201-32FH-1#install apply synchronous
Tue Aug 24 19:47:46.258 UTC
Once the packaging dependencies have been determined, the install operation may have to reload the system.
If you want more control of the operation, then explicitly use 'install apply restart' or 'install apply reload' as reported by 'show install request'.
Continue? [yes/no]:[yes] yes
Starting:
  install apply restart
Atomic change 1.1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing
Current activity: Apply by restarting processes ......

Atomic change 1.1: 'install apply restart' completed without error
RP/0/RP0/CPU0:8201-32FH-1#
</code>
</pre>
</div>

After this operation, multicast package is not active anymore:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active summary
Tue Aug 24 19:51:36.254 UTC
Active Packages:    XR: 176    All: 1272
Label:              7.3.15
Software Hash:      d6032915312fb30abb94375a4720a133

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
xr-8000-netflow                                                   7.3.15v1.0.0-1
xr-bgp                                                            7.3.15v1.0.0-1
xr-ipsla                                                          7.3.15v1.0.0-1
xr-is-is                                                          7.3.15v1.0.0-1
xr-lldp                                                           7.3.15v1.0.0-1
xr-mpls-oam                                                       7.3.15v1.0.0-1
xr-netflow                                                        7.3.15v1.0.0-1
xr-ospf                                                           7.3.15v1.0.0-1
xr-perfmgmt                                                       7.3.15v1.0.0-1
xr-track                                                          7.3.15v1.0.0-1
RP/0/RP0/CPU0:8201-32FH-1#
</code>
</pre>
</div>

This can be confirmed with mrib process absence:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh processes mrib
Tue Aug 24 19:53:16.683 UTC
<mark>No such process mrib</mark>
RP/0/RP0/CPU0:8201-32FH-1#
</code>
</pre>
</div>

The last step is to save current software configuration with *install commit* command. Until this command is executed, package will remain in the committed software configuration and in the repository:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install committed summary
Tue Aug 24 19:52:19.470 UTC
Committed Packages: XR: 178    All: 1274
Label:              7.3.15
Software Hash:      9149973e08793c3f44e84a4a5b385c8a

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
<mark>xr-8000-mcast                                                     7.3.15v1.0.0-1</mark>
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

RP/0/RP0/CPU0:8201-32FH-1#<mark>sh install available</mark>
Tue Aug 24 19:52:37.862 UTC
Trying to access repositories...

Package                                              Architecture                         Version Repository                          Cached
---------------------------------------------------- ---------------- --------------------------- ----------------------------------- ------
xr-8000-mcast                                        x86_64                        7.3.15v1.0.0-1                                     Y
xr-mcast                                             x86_64                        7.3.15v1.0.0-1                                     
</code>
</pre>
</div>

After the commit, active and committed packages are the same and package is no longer present in the repository:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#<mark>install commit synchronous</mark>
Tue Aug 24 19:54:22.029 UTC
Starting:
  install commit
Transaction 1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing
Current activity: Commit transaction .

Transaction 1: 'install commit' completed without error
RP/0/RP0/CPU0:8201-32FH-1#

RP/0/RP0/CPU0:8201-32FH-1#sh install committed summary
Tue Aug 24 19:55:12.740 UTC
Committed Packages: XR: 176    All: 1272
Label:              7.3.15
Software Hash:      d6032915312fb30abb94375a4720a133

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
xr-8000-netflow                                                   7.3.15v1.0.0-1
xr-bgp                                                            7.3.15v1.0.0-1
xr-ipsla                                                          7.3.15v1.0.0-1
xr-is-is                                                          7.3.15v1.0.0-1
xr-lldp                                                           7.3.15v1.0.0-1
xr-mpls-oam                                                       7.3.15v1.0.0-1
xr-netflow                                                        7.3.15v1.0.0-1
xr-ospf                                                           7.3.15v1.0.0-1
xr-perfmgmt                                                       7.3.15v1.0.0-1
xr-track                                                          7.3.15v1.0.0-1

RP/0/RP0/CPU0:8201-32FH-1#sh install available
Tue Aug 24 19:55:49.929 UTC
Trying to access repositories...
<mark>No matching packages found.</mark>
RP/0/RP0/CPU0:8201-32FH-1#
</code>
</pre>
</div>

Package has been fully deactivated and removed from the router.

**Info:** Note as software configuration has been updated, Software Hash value has also changed.
{: .notice--primary}

## Conclusion
This article covered how to remove a package from a Cisco 8000 running IOS XR7 architecture. While philosophy remains identical between classic IOS-XR, IOS-XR64bit and IOS XR7 (modular Network Operating Systems, ability to add and remove packages or software patches), few operational differences exist to deactivate and remove a package.
