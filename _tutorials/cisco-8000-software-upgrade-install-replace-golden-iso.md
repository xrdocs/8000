---
published: true
date: '2024-07-03 16:01 +0200'
title: Cisco 8000 Software Upgrade with install replace & Golden ISO
author: Frederic Cuiller
excerpt: >-
  This article describes how install replace procedure & Golden ISO file can be
  used to stage, patch, or upgrade the Cisco IOS XR Network Operation System
  (NOS) on a Cisco 8000.
tags:
  - Cisco 8000
  - install replace
  - Golden ISO
  - Software Upgrade
position: hidden
---
{% include toc icon="table" title="Cisco 8000 Software Upgrade using install replace & Golden ISO" %}

# Introduction

This article describes how install replace procedure & Golden ISO file can be used to stage, patch, or upgrade the Cisco IOS XR Network Operation System (NOS) running on a Cisco 8000.  
The Golden ISO concept & creation process is covered in [this article](foobar.com). Once created, the Golden ISO file can be used to perform USB boot, iPXE boot, software staging, SMU installation, or software upgrade.

# install replace Concept

install replace functionality was initially introduced on ASR9000 with IOS XR 64bit 6.5.1. As the name says, it replaces all packages with the given ISO and applies the operation.  

Often described as complex, IOS XR software upgrade involves the following:

- Use multiple steps (install add, activate and commit) for IOS XR 32bit, with a long list of packages arguments
- If a package is present in version N, the same package must be present in version N+1
- Golden ISO upgrade always triggers a system reload on IOS XR 64bit, even if the only thing to do is to add the CDP package.

install replace was since improved on IOS XR7 architecture which Cisco 8000 runs. It aims at making software staging, patching (SMU installation), and upgrade easier: 1 CLI to rule them all.  

A Golden ISO install replace operation will always perform the smallest change possible. For example, if the install replace operation detects a single SMU must be added and activated and this SMU requires to perform a process restart, this is what will happen. The whole system will not be reloaded.

![cisco-8000-giso-replace.png]({{site.baseurl}}/images/cisco-8000-giso-replace.png)

The picture above shows the target software configuration is the same as the content of the GISO. To achieve the desired state, install replace:

- Removes the ISIS package
- Downgrades the PFI package
- Upgrades the AIB package
- Adds the BGP and OSPF packages

**Note:** In case of a software upgrade (or downgrade), the system is still reloaded to boot on the target software release.
{: .notice--info}

# Prerequisite

Before upgrading, the system must be in SIA compliant state. Software Innovation Access (SIA) subscription is required to access to the latest Feature Releases. SMU installation is still possible for a given release if the system is non-compliant.  

The following command allow to check compliancy:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Saturn-8711-32FH-M#sh license platform summary
Tue Jul  2 13:48:07.272 UTC
Collection: LAST: Tue Jul 02 2024 13:13:37 UTC
            NEXT: Tue Jul 02 2024 14:13:37 UTC
Reporting:  LAST: Tue Jul 02 2024 09:13:37 UTC
            NEXT: Wed Jul 03 2024 09:13:37 UTC
<mark>SIA Status: In Compliance</mark>
                                                                   Count
Feature/Area     Entitlement                                     Last Next
============ =================================================== ==== ====
FCM 2 Core/E Core and Edge Premier RTU per 100G for Cisco 8000E     4    4
FCM 2 Core/E 8000 and 8000E Type C Device SIA FCM 2.0               2    2
</code>
</pre>
</div>

If the system is not compliant, install replace procedure will fail as illustrated below:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#show license platform detail
Mon Jul  1 11:31:03.513 UTC
Collection: LAST: Mon Jul 01 2024 11:09:05 UTC
            NEXT: Mon Jul 01 2024 12:09:05 UTC
Reporting:  LAST: Sun Jun 30 2024 19:09:05 UTC
            NEXT: Mon Jul 01 2024 19:09:05 UTC
**********************************IMPORTANT************************************
<mark>SIA Status: Out of Compliance(Grace Period Expired)
            SW Upgrades are blocked as SIA grace period has expired</mark>
*******************************************************************************
Parameters: Collection interval:          60 minute(s)
            Reporting  interval:        1440 minute(s)
            Throughput gauge:        1000000 Kbps

RP/0/RP0/CPU0:8202-32FH-M_27#install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso synchronous
Mon Jul  1 11:27:33.048 UTC
Once the packaging dependencies have been determined, the install operation may have to reload the system.
If you want to control the timing of system reload, you must not continue, but use the 'install package replace' command instead, followed by 'install apply'.
Continue? [yes/no]:[yes] yes
Starting:
  install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso
Atomic change 1.1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing ..
Current activity: Verify input and download ISO .........
Current activity: Extract main ISO contents to internal repository if needed^@ ............^@..
Current activity: Veto check ...
<mark>RP/0/RP0/CPU0:Jul  1 11:30:17.193 UTC: smart_lic_sia[65989]: %LICENSE-PLAT_CLIENT-2-SIA_EVAL_EXPIRED : SW Upgrades are blocked as SIA grace period has expired</mark>
RP/0/RP0/CPU0:Jul  1 11:30:19.968 UTC: instorch[147]: %INFRA-INSTALL-3-VETO : Install operation has been vetoed by 1 install clients. First veto is from 0/RP0/CPU0 on plat_sl_client in process 369: CHECK: Unable to proceed. Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai
RP/0/RP0/CPU0:Jul  1 11:30:19.980 UTC: instorch[147]: %INFRA-INSTALL-3-ACTION_FAILED : Packaging operation 1.1.1 failed - Operation vetoed by 1 install client. Error is from plat_sl_client on 0/RP0/CPU0: CHECK: Unable to proceed. Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai
RP/0/RP0/CPU0:Jul  1 11:30:19.980 UTC: instorch[147]: %INFRA-INSTALL-3-ACTION_FAILED : Install operation (Packaging operation) 1.1.1 failed - Operation vetoed by 1 install client. Error is from plat_sl_client on 0/RP0/CPU0: CHECK: Unable to proceed. Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai
.....^@.

<mark>Atomic change 1.1: 'install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso' failed - Operation vetoed by 1 install client. Error is from plat_sl_client on 0/RP0/CPU0: CHECK: Unable to proceed.
Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai. Query 'show install history id 1.1.1 errors location 0/RP0/CPU0' for more details and next steps. Automatically recovered after failure, ready for next user request.</mark>
RP/0/RP0/CPU0:8202-32FH-M_27#
</code>
</pre>
</div>

It’s strongly recommended to read the documentation available on cisco.com. Each IOS XR software release is shipped with a System Upgrade Procedure. This document is available in the 8000-x64-release.docs.tar file as shown below
  
![cisco-8000-rtfm.png]({{site.baseurl}}/images/cisco-8000-rtfm.png)

The document provides useful information on supported upgrade/downgrade paths, if some prerequisite SMU are needed or not, list of caveats, etc.

# Initial State

For this article, a Cisco 8200 is upgraded from IOS XR 7.10.1 to 24.2.1.  
The initial software configuration can be captured using ‘show install active summary’ command. It lists all active packages and SMU:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh install active summary
Mon Jul  1 11:24:11.903 UTC
Active Packages:    XR: 206    All: 1537
<mark>Label:              7.10.1</mark>
Software Hash:      0a617ec67042adfba74f04f893e7fa0c30b1f355de4837f5cd52439d15100605

Optional Packages                                                        <mark>Version</mark>
---------------------------------------------------- ---------------------------
xr-8000-l2mcast                                                   <mark>7.10.1v1.0.0-1</mark>
xr-8000-mcast                                                     7.10.1v1.0.0-1
xr-8000-netflow                                                   7.10.1v1.0.0-1
xr-bgp                                                            7.10.1v1.0.0-1
xr-ipsla                                                          7.10.1v1.0.0-1
xr-is-is                                                          7.10.1v1.0.0-1
xr-lldp                                                           7.10.1v1.0.0-1
xr-mcast                                                          7.10.1v1.0.0-1
xr-mpls-oam                                                       7.10.1v1.0.0-1
xr-netflow                                                        7.10.1v1.0.0-1
xr-ops-script-repo                                                7.10.1v1.0.0-1
xr-ospf                                                           7.10.1v1.0.0-1
xr-perf-meas                                                      7.10.1v1.0.0-1
xr-perfmgmt                                                       7.10.1v1.0.0-1
xr-track                                                          7.10.1v1.0.0-1
RP/0/RP0/CPU0:8202-32FH-M_27#
</code>
</pre>
</div>

In nominal state, the active and committed software configuration must be identical. This can be quicky confirmed checking the software hash value which must match on both outputs:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh install committed summary
Mon Jul  1 11:24:30.949 UTC
Committed Packages: XR: 206    All: 1537
Label:              7.10.1
<mark>Software Hash:      0a617ec67042adfba74f04f893e7fa0c30b1f355de4837f5cd52439d15100605</mark>

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
xr-8000-l2mcast                                                   7.10.1v1.0.0-1
xr-8000-mcast                                                     7.10.1v1.0.0-1
xr-8000-netflow                                                   7.10.1v1.0.0-1
xr-bgp                                                            7.10.1v1.0.0-1
xr-ipsla                                                          7.10.1v1.0.0-1
xr-is-is                                                          7.10.1v1.0.0-1
xr-lldp                                                           7.10.1v1.0.0-1
xr-mcast                                                          7.10.1v1.0.0-1
xr-mpls-oam                                                       7.10.1v1.0.0-1
xr-netflow                                                        7.10.1v1.0.0-1
xr-ops-script-repo                                                7.10.1v1.0.0-1
xr-ospf                                                           7.10.1v1.0.0-1
xr-perf-meas                                                      7.10.1v1.0.0-1
xr-perfmgmt                                                       7.10.1v1.0.0-1
xr-track                                                          7.10.1v1.0.0-1
RP/0/RP0/CPU0:8202-32FH-M_27#
</code>
</pre>
</div>

Like any software upgrade, both IOS XR and Field Programmable Device (FPD) are upgraded. While FPD auto-upgrade is enabled by default on Cisco 8000, having it the configuration allows self-documentation:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:Saturn-8711-32FH-M(config)#fpd auto-upgrade ?
  disable  Disable fpd auto upgrade
  enable   Enable fpd auto upgrade
  exclude  FPD auto-upgrade exclude configuration
  include  FPD auto-upgrade exclude configuration
RP/0/RP0/CPU0:Saturn-8711-32FH-M(config)#fpd auto-upgrade enable
</code>
</pre>
</div>

**Note:** there is no optional FPD package anymore on Cisco 8000. Both xr-fpd, xr-8000-fpd are natively part of the ISO/GISO.
{: .notice--info}


**Note:** some FPD are not covered by FPD auto-upgrade feature. Always check FPD versions after system upgrade and reload.
{: .notice--info}

FPD versions can be checked with following command:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh hw-module fpd
Mon Jul  1 11:24:45.284 UTC

<mark>Auto-upgrade:Enabled</mark>
Attribute codes: B golden, P protect, S secure, A Anti Theft aware
                                                                         FPD Versions
                                                                        ==============
Location   Card type             HWver FPD device       ATR Status   Running Programd  Reload Loc
-------------------------------------------------------------------------------------------------
0/RP0/CPU0 8202-32FH-M           1.0   Bios             S   CURRENT    1.09    1.09    0/RP0/CPU0
0/RP0/CPU0 8202-32FH-M           1.0   BiosGolden       BS  CURRENT            1.01    0/RP0/CPU0
0/RP0/CPU0 8202-32FH-M           1.0   DbIoFpga1            CURRENT    1.06    1.06         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   DbIoFpgaGolden1  B   CURRENT            1.03         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   IoFpga1              CURRENT    1.06    1.06         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   IoFpgaGolden1    B   CURRENT            1.03         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   SsdMicron5300    S   CURRENT    0.01    0.01         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   x86Fpga          S   CURRENT    1.07    1.07         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   x86FpgaGolden    BS  CURRENT            1.03         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   x86TamFw         S   CURRENT    7.12    7.12         0/RP0
0/RP0/CPU0 8202-32FH-M           1.0   x86TamFwGolden   BS  CURRENT            7.10         0/RP0
0/PM0      PSU2KW-ACPI           0.0   PO-PrimMCU           CURRENT    1.03    1.03       NOT REQ
0/PM0      PSU2KW-ACPI           0.0   PO-SecMCU            CURRENT    1.08    1.08       NOT REQ
0/PM1      PSU2KW-ACPI           0.0   PO-SecMCU            CURRENT    1.08    1.08       NOT REQ
0/PM1      PSU2KW-ACPI           0.0   PrimMCU              NOT READY                         N/A
0/FB0      8202-32FH-M[FB]       1.0   IoFpga               CURRENT    1.10    1.10       NOT REQ
0/FB0      8202-32FH-M[FB]       1.0   IoFpgaGolden     B   CURRENT            1.00       NOT REQ
RP/0/RP0/CPU0:8202-32FH-M_27#
</code>
</pre>
</div>

# Pre-upgrade Tasks

The first step is to copy the Golden ISO file on the router. Having the GISO available locally could be useful for certain situations (e.g copy the file to a USB stick to perform USB boot on another router, iPXE boot, disaster recovery).  

Once copied, file integrity must be checked:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh md5 file /harddisk:/8000-golden-x86_64-24.1.2-$
Mon Jul  1 11:21:05.352 UTC
ad8cc593ab4cba2f854b4dcf732d0547
</code>
</pre>
</div>

To double check software upgrade pre-requisites, it’s possible to leverage the upgrade matrix. This feature introduced in IOS XR 7.5.2 helps to determine supported upgrade or downgrade paths.  

In this example, target GISO is provided as an argument. The system will check current software configuration and verify if the upgrade path to the target GISO release is supported, or if it requires SMU installation for instead:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#show install upgrade-matrix iso /harddisk:/8000-golden-x86_64-24.1.2-1337.iso
Mon Jul  1 11:21:52.195 UTC
-------------------------------------------------------------
Upgrade matrix information for system upgrade: <mark>7.10.1->24.1.2</mark>
-------------------------------------------------------------

<mark>XR system upgrade is supported with no additional restrictions</mark>
</code>
</pre>
</div>

When used with the ‘all’ keyword, upgrade matrix will display all upgrade and downgrade configurations:  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#show install upgrade-matrix iso /harddisk:/8000-golden-x86_64-24.1.2-1337.iso all
Mon Jul  1 11:23:00.596 UTC
Matrix: XR version: 24.1.2, File version: 1.0, Version: N/A

The upgrade matrix indicates that the following system upgrades are supported:

From       To         Restrictions
---------- ---------- ----------------------------------------------------
24.1.1     24.1.2     -
24.1.2     24.1.1     -
24.1.2     7.10.1     -
24.1.2     7.10.2     -
24.1.2     7.11.1     -
24.1.2     7.11.2     -
24.1.2     7.5.3      Target fixes; Caveats; Replace performed via reimage
24.1.2     7.5.4      Target fixes; Caveats; Replace performed via reimage
24.1.2     7.5.5      Caveats; Replace performed via reimage
24.1.2     7.7.1      Target fixes; Caveats; Replace performed via reimage
24.1.2     7.7.2      Target fixes; Caveats; Replace performed via reimage
24.1.2     7.8.1      Target fixes; Caveats; Replace performed via reimage
24.1.2     7.8.2      Target fixes; Caveats; Replace performed via reimage
24.1.2     7.9.1      Caveats; Replace performed via reimage
24.1.2     7.9.2      Caveats; Replace performed via reimage
7.10.1     24.1.2     -
7.10.2     24.1.2     -
7.11.1     24.1.2     -
7.11.2     24.1.2     -
7.3.3      24.1.2     Bridging fixes; Caveats
7.3.4      24.1.2     Bridging fixes; Caveats
7.5.3      24.1.2     Caveats
7.5.4      24.1.2     Caveats
7.5.5      24.1.2     Caveats
7.7.1      24.1.2     Caveats
7.7.2      24.1.2     Caveats
7.8.1      24.1.2     Caveats
7.8.2      24.1.2     Caveats
7.9.1      24.1.2     Caveats
7.9.2      24.1.2     Caveats

Add the from and to versions to the end of the CLI command, for data on versions with additional restrictions

For example, to display restrictions for the 24.1.1->24.1.2 upgrade, use
        'show install upgrade-matrix iso /harddisk:/8000-golden-x86_64-24.1.2-1337.iso 24.1.1 24.1.2'
</code>
</pre>
</div>

The next step of the preparation is to ensure system is stable & healthy before the upgrade. A sample list is given as an example below but should be customized for each customer and deployment (e.g if the router runs multicast or SRv6, extra health check should be performed):

<div class="highlighter-rouge">
<pre class="highlight">
<code>
terminal length 0
terminal width 0

show install active summary
show install committed summary
show install available
show platform
show led
show inventory
show configuration commit list
show environment all
show media location all
show filesystem location 0/rp0/CPU0
show filesystem location 0/rp1/CPU0	
show processes blocked
show processes blocked
show redundancy
show hw-module fpd

show process cpu | ex 0%      0%       0%
sh drops all location all
sh cef drop

show run
show run formal | utility sort
show logging
show interfaces description | ex admin-down
show interfaces description | inc "up          up" | utility wc
show interfaces
show ipv4 interface brief
show ipv6 interface brief

show interface description | inc BE[0-9]+
show bundle
show bundle brief
show lacp counters

show ospf neighbor
show ospf neighbor detail
show ospf neighbor | inc FULL | utility wc

sh isis neighbors
sh isis neighbors detail
sh isis neighbors | i Total neighbor
       
show mpls interfaces
show mpls ldp neighbor
show mpls ldp neighbor brief
show mpls ldp neighbor brief | utility wc

show bfd all session
show bfd all session | inc UP | utility wc

show route vrf all summary
sh bgp instance all ipv4 unicast summary
sh bgp instance all vpnv4 unicast summary

sh bgp instance all ipv4 unicast neighbor
sh bgp instance all vpnv4 unicast neighbor

show route vrf all

copy running-config hardisk:backup-pre.conf
</code>
</pre>
</div>

Once the sanity check is done, it’s recommended to drain/isolate the router. This can be achieved using regular routing techniques (ISIS overload bit, OSPF max metric, MPLS traffic engineering, BGP attributes, etc.).

# IOS XR Upgrade using install replace

Software upgrade can be launched using ‘install replace’ command. install replace comes with several options:
- commit will commit software configuration after system reload. This means there is no way to rollback to previous release by doing a system reload.
- iso-config option can be used to replace the current system configuration by the configuration present in the Golden ISO. This option can be interesting for system staging and load a vanilla config
- while install replace always makes the smallest possible change, it’s possible to tell the system to reload

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso ?
  asynchronous                Install operation will proceed asynchronously with output to console
  commit                      Additionally commit the software
  force                       Ignore non-critical errors
  iso-config                  Specify handling of ISO config
  noprompt                    Install operation will proceed with no prompting
  reload                      Install apply will proceed via a reload (even if not necessary)
  skip-upgrade-matrix-checks  Skip upgrade matrix checks when moving between XR versions. Note: skipping the check risks performing an un-certified upgrade, which is not guaranteed to succeed
  synchronous                 Install operation will proceed synchronously
                              Install operation will proceed with no output to console
</code>
</pre>
</div>

