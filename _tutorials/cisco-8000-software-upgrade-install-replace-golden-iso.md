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
The Golden ISO concept & creation process is covered in [this article]({{site.baseurl}}/tutorials/8000-software-xr7-giso). Once created, the Golden ISO file can be used to perform USB boot, iPXE boot, software staging, SMU installation, or software upgrade.

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

In details, the process takes:

- 12min to extract the GISO and add the packages in the local repository

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#<mark>install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso synchronous</mark>
Mon Jul  1 11:36:41.803 UTC
Once the packaging dependencies have been determined, the install operation may have to reload the system.
If you want to control the timing of system reload, you must not continue, but use the 'install package replace' command instead, followed by 'install apply'.
Continue? [yes/no]:[yes] yes
Starting:
  install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso
Atomic change 2.1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing ..
Current activity: Verify input and download ISO .........
Current activity: Extract main ISO contents to internal repository if needed ...........^@.
Current activity: Veto check ....
Current activity: Package add or other package operation ....^@...........^@............^@............^@...........^@............^@...........^@............^@...........^@.........
Current activity: Apply by reload

<mark>Atomic change 2.1: Pre-reload steps completed without error, reloading</mark>
</code>
</pre>
</div>

- 6min to reload on the target release

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:Jul  1 11:48:37.826 UTC: instorch[147]: %INFRA-INSTALL-2-SYSTEM_RELOAD : As part of an install operation, a full system reload will take place in order to apply the installed software.</mark>
.
Preparing system for backup. This may take a few minutes especially for large configurations.
	Status report: node0_RP0_CPU0: START TO BACKUP
	Status report: node0_RP0_CPU0: BACKUP HAS COMPLETED SUCCESSFULLY
[Done]
.^@.......RP/0/RP0/CPU0:Jul  1 11:49:22.262 UTC: shelfmgr[232]: %PLATFORM-CPA_INTF_SHELFMGR-3-CARD_RELOAD_FAILED : Failed to perform card reload operation on card: node0_RP0_CPU0, err: 0
[  OK  ] Stopped IOS-XR Reaperd and Process Manager.
         Stopping Cisco Directory Services...
[  OK  ] Stopped NOS Bootup FPD Upgrade Service.
         Stopping Lightning Fast We…h Light System Requirements...
[  OK  ] Stopped Lightning Fast Web…ith Light System Requirements.
[  OK  ] Stopped Cisco Directory Services.
[  OK  ] Stopped Setup Network OS Bootstrap.
         Stopping CPA Setup...
[  OK  ] Stopped NPU SDK Setup.
         Stopping IOS-XR ISO Installation including sshd...
[  OK  ] Stopped CPA Setup.
         Stopping FPGA Setup...
[  OK  ] Stopped FPGA Setup.
[  OK  ] Stopped IOS-XR ISO Installation including sshd.
         Stopping Service for factory reset...
[  OK  ] Stopped OpenSSH Key Generation.
[  OK  ] Stopped Service for factory reset.
[  OK  ] Stopped target Basic System.
[  OK  ] Stopped target Paths.
[  OK  ] Stopped target Slices.
[  OK  ] Removed slice User and Session Slice.
[  OK  ] Stopped target Sockets.
[  OK  ] Closed D-Bus System Message Bus Socket.
[  OK  ] Closed sshd.socket.
[  OK  ] Stopped target System Initialization.
[  OK  ] Closed Syslog Socket.
         Stopping Board Setup...
[  OK  ] Stopped Apply Kernel Variables.
[  OK  ] Stopped Load Kernel Modules.
         Stopping Update UTMP about System Boot/Shutdown...
[  OK  ] Stopped Update UTMP about System Boot/Shutdown.
         Stopping Security Auditing Service...
[  OK  ] Stopped Security Auditing Service.
[  OK  ] Stopped Create Volatile Files and Directories.
[  OK  ] Stopped target Local File Systems.
         Unmounting /boot/efi...
         Unmounting /mnt/dr_part...
         Unmounting /mnt/fuse/ftp...
         Unmounting /mnt/fuse/nvgen_server...
         Unmounting /mnt/fuse/parser_server...
         Unmounting /mnt/fuse/tftp...
         Unmounting /mnt/pdtmpfs...
         Unmounting /qsm...
         Unmounting /run/netns/default...
         Unmounting /run/netns/global-vrf...
         Unmounting /run/netns/vrf-default...
         Unmounting /run/netns/xrnns...
         Unmounting /selinux...
         Unmounting Temporary Directory (/tmp)...
         Unmounting /var/lib/docker...
         Unmounting /var/volatile...
         Unmounting /var/xr/disk1...
         Unmounting /var/xr/scratch...
         Stopping Flush Journal to Persistent Storage...
[  OK  ] Stopped Board Setup.
[  OK  ] Unmounted /boot/efi.
[  OK  ] Unmounted /mnt/dr_part.
[  OK  ] Unmounted /mnt/fuse/ftp.
[  OK  ] Unmounted /mnt/fuse/nvgen_server.
[  OK  ] Unmounted /mnt/fuse/parser_server.
[  OK  ] Unmounted /mnt/fuse/tftp.
[  OK  ] Unmounted /mnt/pdtmpfs.
[  OK  ] Unmounted /qsm.
[  OK  ] Unmounted /run/netns/default.
[  OK  ] Unmounted /run/netns/global-vrf.
[  OK  ] Unmounted /run/netns/vrf-default.
[  OK  ] Unmounted /run/netns/xrnns.
[  OK  ] Unmounted /selinux.
[  OK  ] Unmounted Temporary Directory (/tmp).
[  OK  ] Unmounted /var/lib/docker.
[  OK  ] Unmounted /var/volatile.
[  OK  ] Unmounted /var/xr/scratch.
[  OK  ] Stopped Flush Journal to Persistent Storage.
         Unmounting /boot...
         Unmounting /mnt...
         Unmounting /run/netns...
         Unmounting /var/log...
[  OK  ] Stopped File System Check …ain-xr-vg/install-data-docker.
[  OK  ] Stopped File System Check …in-xr-vg/install-data-scratch.
[  OK  ] Stopped File System Check on /dev/sda1.
[  OK  ] Stopped File System Check on /dev/sda2.
[  OK  ] Unmounted /var/xr/disk1.
[  OK  ] Unmounted /boot.
[  OK  ] Unmounted /mnt.
[  OK  ] Unmounted /run/netns.
[FAILED] Failed unmounting /var/log.
[  OK  ] Stopped target Swap.
[  OK  ] Reached target Unmount All Filesystems.
[  OK  ] Stopped File System Check on /dev/main-xr-vg/boot-lv.
[  OK  ] Stopped File System Check …main-xr-vg/install-data-disk1.
[  OK  ] Stopped target Local File Systems (Pre).
         Stopping Monitoring of LVM…meventd or progress polling...
[  OK  ] Stopped Remount Root and Kernel File Systems.
[  OK  ] Stopped Create Static Device Nodes in /dev.
[  OK  ] Stopped Monitoring of LVM2… dmeventd or progress polling.
         Stopping Device-mapper event daemon...
[  OK  ] Stopped Device-mapper event daemon.
[  OK  ] Reached target Shutdown.
[  OK  ] Reached target Final Step.
[  OK  ] Started Reboot.
[  OK  ] Reached target Reboot.
[319664.699919] systemd-shutdown[1]: Could not detach DM /dev/dm-9: Device or resource busy
[319664.709780] systemd-shutdown[1]: Could not detach DM /dev/dm-8: Device or resource busy
[319664.719635] systemd-shutdown[1]: Could not detach DM /dev/dm-7: Device or resource busy
[319664.744445] systemd-shutdown[1]: Failed to finalize  DM devices, ignoring
[319664.845458] Unsupported TPM Send Cmd! tpm_tag=8001,tpm_ordinal=0145
[319664.853194] tpm tpm0: tpm_try_transmit: send(): error -11
[319665.190333] pm-fpga: Turning Status LED to amber, Active, Alarm, Board and Chassis Attention LED to OFF ...
<mark>[319665.212037] pm-fpga: Power cycling the card via FPGA</mark>



Cisco 8000(R) Series BIOS Ver 1.09 Primary
Intel(R) Xeon(R) CPU D-1530 @ 2.40GHz
Board Type 0x111 PID 8202-32FH-M Serial FLM260902HU
X86FPGA 1.7.0 TamLib 5.01.01
<mark>Booting from Disk
  Booting `IOS-XR-latest'

Booting latest from Disk..</mark>
Loading Kernel..
Verifying /bzImage...
/bzImage verified using attached signature.
Verifying /bzImage...
/bzImage verified using attached signature.
Loading initrd..
Verifying /initrd.img...
/initrd.img verified using Pkcs7 signature.
[   11.389573] Created proc for bigphysarea
Starting version 244.5+
Located disaster recovery partition /dev/sda2
  Found volume group "main-xr-vg" using metadata type lvm2
  9 logical volume(s) in volume group "main-xr-vg" now active
  9 logical volume(s) in volume group "main-xr-vg" now active
  9 logical volume(s) in volume group "main-xr-vg" now active
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Boot: 34/61184 files (8.8% non-contiguous), 34986/244736 blocks
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Disk1: 281/3407872 files (5.7% non-contiguous), 2197946/13612032 blocks
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Docker: 47/378256 files (8.5% non-contiguous), 59648/1512448 blocks
fsck from util-linux 2.35.1
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Log: 14454/303104 files (0.7% non-contiguous), 142168/1210368 blocks
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
Scratch: 72/218160 files (6.9% non-contiguous), 33788/871424 blocks
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
RootFS: 102303/1515520 files (0.5% non-contiguous), 4473288/12099584 blocks
fsck from util-linux 2.35.1
e2fsck 1.45.7 (28-Jan-2021)
fsck.ext2: No such file or directory while trying to open /dev/main-xr-vg/install-rootfs-snapshot-1719833805
Possibly non-existent device?
Starting kdump:

kernel.panic_on_oops = 1
fstab modified mount options for 1 partitions
Setup encrypted LV...
Execute: cryptsetup luksOpen /dev/main-xr-vg/install-data-encrypted_in encrypted -d '-'
Invoke selinux initialization...
Enable selinux to relabel filesystem from initramfs
Loading custom SELinux policy modules...
iosxr policy module loaded
Checking SELinux security contexts:
 * First booting, filesystem will be relabeled...
 * No need reboot the system.
SELinux relabel complete
Setting up IMA...
Searching for IMA trusted keyring
->IMA trusted keyring found
Importing IMA x509 certificate into the IMA trusted keyring
->IMA certificate found: /etc/keys/dbCisco-4.der (RELEASE)
->Certificate imported into IMA keyring
Loading IMA policy
IMA policy for production
IMA policy loaded successfully!
->Secure Boot CA certificate found: /etc/keys/dbCisco-1.der (RELEASE)
->Certificate imported into LNT user keyring
Generate audit rules from component rules
/sbin/augenrules: No change
Switch to new root and run init

Welcome to Linux Distro for XR 11.1.2 (dunfell)!

[  OK  ] Created slice system-serial\x2dgetty.slice.
[  OK  ] Created slice system-systemd\x2dfsck.slice.
[  OK  ] Created slice User and Session Slice.
[  OK  ] Reached target Paths.
[  OK  ] Reached target Remote File Systems.
[  OK  ] Reached target Slices.
[  OK  ] Reached target Swap.
[  OK  ] Listening on Device-mapper event daemon FIFOs.
[  OK  ] Listening on RPCbind Server Activation Socket.
[  OK  ] Reached target RPC Port Mapper.
[  OK  ] Listening on Syslog Socket.
[  OK  ] Listening on Journal Audit Socket.
[  OK  ] Listening on Journal Socket (/dev/log).
[  OK  ] Listening on Journal Socket.
[  OK  ] Listening on Network Service Netlink Socket.
[  OK  ] Listening on udev Control Socket.
[  OK  ] Listening on udev Kernel Socket.
         Mounting Huge Pages File System...
         Mounting POSIX Message Queue File System...
         Mounting Temporary Directory (/tmp)...
         Starting Availability of block devices...
         Starting Create list of st…odes for the current kernel...
         Starting Monitoring of LVM…meventd or progress polling...
         Starting RPC Bind...
         Starting SELinux init for /dev service loading...
         Starting Board Setup...
         Starting Journal Service...
         Starting Load Kernel Modules...
         Starting Remount Root and Kernel File Systems...
         Starting udev Coldplug all Devices...
[  OK  ] Started RPC Bind.
[  OK  ] Mounted Huge Pages File System.
[  OK  ] Mounted POSIX Message Queue File System.
[  OK  ] Mounted Temporary Directory (/tmp).
[  OK  ] Started Availability of block devices.
[  OK  ] Started Create list of sta… nodes for the current kernel.
[  OK  ] Started Monitoring of LVM2… dmeventd or progress polling.
[  OK  ] Started SELinux init for /dev service loading.
[  OK  ] Started Board Setup.
[  OK  ] Started Journal Service.
[  OK  ] Started Load Kernel Modules.
[  OK  ] Started Remount Root and Kernel File Systems.
         Mounting FUSE Control File System...
         Starting Rebuild Hardware Database...
         Starting Apply Kernel Variables...
         Starting Create System Users...
[  OK  ] Mounted FUSE Control File System.
[  OK  ] Started Apply Kernel Variables.
[  OK  ] Started udev Coldplug all Devices.
[  OK  ] Started Create System Users.
         Starting Create Static Device Nodes in /dev...
[  OK  ] Started Create Static Device Nodes in /dev.
[  OK  ] Reached target Local File Systems (Pre).
         Mounting /mnt...
[  OK  ] Reached target Containers.
         Mounting /var/volatile...
[  OK  ] Mounted /mnt.
[  OK  ] Mounted /var/volatile.
         Starting Load/Save Random Seed...
[  OK  ] Started Load/Save Random Seed.
[  OK  ] Started Rebuild Hardware Database.
         Starting udev Kernel Device Manager...
[  OK  ] Started udev Kernel Device Manager.
[  OK  ] Found device /dev/ttyS1.
[   59.265111] ixgbe 0000:0b:00.0: HW Init failed: -17
[  OK  ] Found device Micron_5300_MTFDDAV480TDS partition01.
[  OK  ] Found device Micron_5300_MTFDDAV480TDS DR-Part.
[  OK  ] Found device /dev/main-xr-vg/install-data-docker.
[  OK  ] Found device /dev/main-xr-vg/install-data-scratch.
[  OK  ] Found device /dev/main-xr-vg/install-data-disk1.
[  OK  ] Found device /dev/main-xr-vg/install-data-log.
[  OK  ] Found device /dev/main-xr-vg/boot-lv.
[  OK  ] Created slice system-lvm2\x2dpvscan.slice.
         Starting LVM event activation on device 8:3...
         Starting File System Check on /dev/main-xr-vg/boot-lv...
         Starting File System Check…in-xr-vg/install-data-disk1...
         Starting File System Check…n-xr-vg/install-data-docker...
         Starting File System Check…main-xr-vg/install-data-log...
         Starting File System Check…-xr-vg/install-data-scratch...
         Starting File System Check on /dev/sda1...
         Starting File System Check on /dev/sda2...
[  OK  ] Started LVM event activation on device 8:3.
[  OK  ] Started File System Check on /dev/sda1.
[  OK  ] Started File System Check on /dev/sda2.
         Mounting /mnt/dr_part...
[  OK  ] Started File System Check on /dev/main-xr-vg/boot-lv.
         Mounting /boot...
[  OK  ] Started File System Check …ain-xr-vg/install-data-docker.
         Mounting /var/lib/docker...
[  OK  ] Started File System Check …in-xr-vg/install-data-scratch.
[  OK  ] Mounted /boot.
         Mounting /boot/efi...
         Mounting /var/xr/scratch...
[  OK  ] Mounted /mnt/dr_part.
[  OK  ] Mounted /var/lib/docker.
[  OK  ] Mounted /var/xr/scratch.
[  OK  ] Mounted /boot/efi.
[  OK  ] Started File System Check …main-xr-vg/install-data-disk1.
         Mounting /var/xr/disk1...
[  OK  ] Mounted /var/xr/disk1.
[  OK  ] Started File System Check …v/main-xr-vg/install-data-log.
         Mounting /var/log...
[  OK  ] Mounted /var/log.
[  OK  ] Reached target Local File Systems.
         Starting SELinux init service loading...
         Starting Flush Journal to Persistent Storage...
         Starting Copy selected logs to var/log/old directories...
[  OK  ] Started SELinux init service loading.
[  OK  ] Started Copy selected logs to var/log/old directories.
[  OK  ] Started Flush Journal to Persistent Storage.
         Starting Create Volatile Files and Directories...
[  OK  ] Started Create Volatile Files and Directories.
         Starting Security Auditing Service...
         Starting Rebuild Journal Catalog...
[  OK  ] Started Security Auditing Service.
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Rebuild Journal Catalog.
[  OK  ] Started Update UTMP about System Boot/Shutdown.
         Starting Update is Completed...
[  OK  ] Started Update is Completed.
[  OK  ] Reached target System Initialization.
[  OK  ] Started Discard unused blocks.
[  OK  ] Started Periodic rotation of log files.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Reached target Timers.
[  OK  ] Listening on D-Bus System Message Bus Socket.
         Starting sshd.socket.
[  OK  ] Listening on sshd.socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
[  OK  ] Started Job spooling tools.
[  OK  ] Started Periodic Command Scheduler.
[  OK  ] Started D-Bus System Message Bus.
         Starting Ethernet Bridge Filtering Tables...
[  OK  ] Started Service for factory reset.
[  OK  ] Started HWID Rescan/Remove Client.
         Starting IPv6 Packet Filtering Framework...
         Starting IPv4 Packet Filtering Framework...
[  OK  ] Started Machine Check Exception Logging Daemon.
[  OK  ] Started Cisco Process Monitor Service.
         Starting Rollback uncommit… config change transactions...
[  OK  ] Started PCIE Link Monitor.
         Starting FPGA Setup...
[  OK  ] Started Self Monitoring an…ing Technology (SMART) Daemon.
[  OK  ] Started System Logging Service.
         Starting Login Service...
         Starting Thermal Daemon Service...
         Starting IOS-XR Setup Non-Root related tasks...
         Starting OpenSSH Key Generation...
         Starting Rotate log files...
[  OK  ] Started Ethernet Bridge Filtering Tables.
[  OK  ] Started IPv6 Packet Filtering Framework.
[  OK  ] Started IPv4 Packet Filtering Framework.
[  OK  ] Started Rollback uncommitt…rk config change transactions.
[  OK  ] Reached target Network (Pre).
         Starting Network Service...
[  OK  ] Started OpenSSH Key Generation.
         Starting IOS-XR ISO Installation including sshd...
[  OK  ] Started IOS-XR Setup Non-Root related tasks.
[  OK  ] Started Login Service.
[  OK  ] Started Network Service.
         Starting Network Name Resolution...
[  OK  ] Started Rotate log files.
[  OK  ] Started Thermal Daemon Service.
[  OK  ] Started Network Name Resolution.
[  OK  ] Reached target Network.
[  OK  ] Reached target Host and Network Name Lookups.
         Starting containerd container runtime...
[  OK  ] Started Respond to IPv6 Node Information Queries.
[  OK  ] Started Network Router Discovery Daemon.
         Starting Permit User Sessions...
[  OK  ] Started Xinetd A Powerful Replacement For Inetd.
[  OK  ] Started Permit User Sessions.
[  OK  ] Started Serial Getty on ttyS1.
[  OK  ] Reached target Login Prompts.
[  OK  ] Started containerd container runtime.
[  OK  ] Started FPGA Setup.
         Starting FPGA Shutdown...
         Starting msixd start...
         Starting Marvell Switch Setup...
         Starting CPA Setup...
[  OK  ] Started Marvell Switch Setup.
[  OK  ] Started msixd start.
[  OK  ] Started FPGA Shutdown.
[  OK  ] Started CPA Setup.
[   62.946593] xrnginstall[1570]: 2024-07-01 11:51:47.421 UTC: xrinstall: Setting up dumper and build info files
[   63.237449] xrnginstall[1800]: 2024-07-01 11:51:47.711 UTC: xrinstall: XR Lineup:  r241x.lu%EFR-00000459055
<mark>[   63.240508] xrnginstall[1804]: 2024-07-01 11:51:47.715 UTC: xrinstall: XR Version: 24.1.2</mark>
[   63.243965] xrnginstall[1808]: 2024-07-01 11:51:47.718 UTC: xrinstall: Completed set up of dumper and build info files
[   63.247333] xrnginstall[1812]: 2024-07-01 11:51:47.722 UTC: xrinstall: Preparing IOS-XR
[   63.254233] xrnginstall[1820]: 2024-07-01 11:51:47.728 UTC: xrinstall: Switching to latest FS
[   69.759903] xrnginstall[1831]: 2024-07-01 11:51:54.234 UTC: xrinstall: Completed switch to latest FS
[   70.353305] xrnginstall[2160]: 2024-07-01 11:51:54.828 UTC: xrinstall: Load platform kernel modules
[   70.467690] xrnginstall[2212]: 2024-07-01 11:51:54.942 UTC: xrinstall: Succeeded loading platform kernel modules
[   70.472330] xrnginstall[2217]: 2024-07-01 11:51:54.947 UTC: xrinstall: Enable hardware
[   71.299312] xrnginstall[2343]: 2024-07-01 11:51:55.773 UTC: xrinstall: Succeeded enabling hardware
[   71.307606] xrnginstall[2352]: 2024-07-01 11:51:55.782 UTC: xrinstall: Setting up owner and partner RPM symlinks
[   71.320853] xrnginstall[2362]: 2024-07-01 11:51:55.795 UTC: xrinstall: Checking if rollback cleanup is required
[   71.324907] xrnginstall[2367]: 2024-07-01 11:51:55.799 UTC: xrinstall: Finished rollback cleanup stage
[   71.332113] xrnginstall[2373]: 2024-07-01 11:51:55.806 UTC: xrinstall: Single node: starting XR
[   71.460094] xrnginstall[2465]: 2024-07-01 11:51:55.934 UTC: xrinstall: xrnginstall completed successfully
RP/0/RP0/CPU0:Jul  1 11:52:22.483 UTC: fpd_client[392]: %PLATFORM-CPA_INTF_FPD-3-ACCESS_ERROR : Node 0/PM1 FPD PrimMCU ACCESS failure ''
RP/0/RP0/CPU0:Jul  1 11:52:32.723 UTC: envmon[327]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :Power Module Error (PM_VIN_VOLT_OOR) :DECLARE :0/PM1: PM1 is under HW_VIN_OUT_OF_RANGE condition(input voltage is out of low line range 90V-140V
RP/0/RP0/CPU0:Jul  1 11:52:32.723 UTC: envmon[327]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :Power Module Output Disabled (PM_OUTPUT_DISABLED) :DECLARE :0/PM1: PM1 is under HW_OUTPUT_DISABLED condition.
RP/0/RP0/CPU0:Jul  1 11:52:32.724 UTC: envmon[327]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :Power Module Error (PM_NO_INPUT_DETECTED) :DECLARE :0/PM1: PM1 is under HW_NO_INPUT_DETECTED condition.
RP/0/RP0/CPU0:Jul  1 11:52:32.725 UTC: envmon[327]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :Power Group Redundancy lost :DECLARE :0:


ios con0/RP0/CPU0 is now available





Press RETURN to get started.
RP/0/RP0/CPU0:Jul  1 11:52:41.730 UTC: fpd_client[392]: %PLATFORM-CPA_INTF_FPD-3-ACCESS_ERROR : Node 0/PM1 FPD PrimMCU ACCESS failure ''
RP/0/RP0/CPU0:Jul  1 11:52:41.738 UTC: fpd_client[392]: %PLATFORM-CPA_INTF_FPD-3-ACCESS_ERROR : Node 0/PM1 FPD PrimMCU ACCESS failure ''
RP/0/RP0/CPU0:Jul  1 11:53:14.905 UTC: tmgctrl[419]: %PLATFORM-CLKCTRL_MAIN-4-TIMING_PLL_NEED_FW_UPG : Timing PLL device needs firmware upgrade. Running ver:0x07d8 Packaged ver: :0x07da

This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply third-party
authority to import, export, distribute or use encryption. Importers,
exporters, distributors and users are responsible for compliance with
U.S. and local country laws. By using this product you agree to comply
with applicable laws and regulations. If you are unable to comply with
U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be
found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.



RP/0/RP0/CPU0:Jul  1 11:53:34.460 UTC: pyztp2[424]: %INFRA-ZTP-4-START : ZTP has started. Interfaces might be brought up if they are shutdown
RP/0/RP0/CPU0:Jul  1 11:53:36.869 UTC: ifmgr[245]: %PKT_INFRA-LINK-3-UPDOWN : Interface MgmtEth0/RP0/CPU0/0, changed state to Down
RP/0/RP0/CPU0:Jul  1 11:53:36.907 UTC: ifmgr[245]: %PKT_INFRA-LINK-3-UPDOWN : Interface FourHundredGigE0/0/0/31, changed state to Down
RP/0/RP0/CPU0:Jul  1 11:53:39.336 UTC: ifmgr[245]: %PKT_INFRA-LINK-3-UPDOWN : Interface MgmtEth0/RP0/CPU0/0, changed state to Up
RP/0/RP0/CPU0:Jul  1 11:53:58.601 UTC: smartlicserver[397]: %LICENSE-SMART_LIC-3-COMM_FAILED : Communications failure with the Cisco Smart License Utility (CSLU) : Unable to resolve server hostname/domain name
RP/0/RP0/CPU0:Jul  1 11:54:06.827 UTC: pyztp2[424]: %INFRA-ZTP-4-EXITED : ZTP exited

User Access Verification
Username:

8202-32FH-M_27 con0/RP0/CPU0 is now available

Press RETURN to get started.
</code>
</pre>
</div>

# Post-upgrade Tasks

After system reload, the new software release can be confirmed:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#show version
Wed Jul  3 13:43:44.552 UTC
Cisco IOS XR Software, Version 24.1.2 LNT
Copyright (c) 2013-2024 by Cisco Systems, Inc.

Build Information:
 Built By     : sajshah
 Built On     : Thu Jun 27 08:07:23 UTC 2024
 Build Host   : iox-ucs-054
 Workspace    : /auto/ioxdepot6/GISO/giso_build_lindt/giso_release_create/fcuiller_2024-06-27_15-04-41_UTC
 <mark>Version      : 24.1.2</mark>
 Label        : 24.1.2-1337

cisco 8000 (Intel(R) Xeon(R) CPU D-1530 @ 2.40GHz)
cisco 8202-32FH-M (Intel(R) Xeon(R) CPU D-1530 @ 2.40GHz) processor with 32GB of memory
8202-32FH-M_27 uptime is 2 days, 1 hour, 53 minutes
Cisco 8200 2RU 32x400G QSFP56-DD w/IOS XR HBM MACsec
</code>
</pre>
</div>

The current active software configuration can be verified:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh install active summary
Mon Jul  1 11:59:03.806 UTC
Active Packages:    XR: 211    All: 1561
<mark>Label:              24.1.2-1337</mark>
Software Hash:      df7dbf1b438fec9acfb21d9d1bc10edc171f4283bbf67bd196d9bfe9331a59fb

Optional Packages                                                        Version
---------------------------------------------------- ---------------------------
xr-8000-l2mcast                                                   24.1.2v1.0.0-1
xr-8000-mcast                                                     24.1.2v1.0.0-1
xr-8000-netflow                                                   24.1.2v1.0.0-1
xr-bgp                                                            24.1.2v1.0.0-1
xr-ipsla                                                          24.1.2v1.0.0-1
xr-is-is                                                          24.1.2v1.0.0-1
xr-k9sec                                                          24.1.2v1.0.0-1
xr-lldp                                                           24.1.2v1.0.0-1
xr-mcast                                                          24.1.2v1.0.0-1
xr-mpls-oam                                                       24.1.2v1.0.0-1
xr-netflow                                                        24.1.2v1.0.0-1
xr-ops-script-repo                                                24.1.2v1.0.0-1
xr-ospf                                                           24.1.2v1.0.0-1
xr-perf-meas                                                      24.1.2v1.0.0-1
xr-perfmgmt                                                       24.1.2v1.0.0-1
xr-track                                                          24.1.2v1.0.0-1
</code>
</pre>
</div>

The Golden ISO label can also be used to verify the right file was used.

At this point, the software configuration is not committed, meaning it’s possible to rollback to IOS XR 7.10.1 by reloading the system:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh install committed summary
Mon Jul  1 11:59:51.743 UTC
Committed Packages: XR: 206    All: 1537
<mark>Label:              7.10.1</mark>
Software Hash:      0a617ec67042adfba74f04f893e7fa0c30b1f355de4837f5cd52439d15100605

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
</code>
</pre>
</div>

The same sanity check is performed and once it’s confirmed system is stable and healthy, software configuration can be committed:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#<mark>install commit synchronous</mark>
Mon Jul  1 12:00:08.881 UTC
Starting:
  install commit
Transaction 2
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing
Current activity: Commit transaction ...

Transaction 2: 'install commit' completed without error
RP/0/RP0/CPU0:8202-32FH-M_27#<mark>sh install committed summary</mark>
Mon Jul  1 12:00:36.425 UTC
Committed Packages: XR: 211    All: 1561
<mark>Label:              24.1.2-1337</mark>
Software Hash:      df7dbf1b438fec9acfb21d9d1bc10edc171f4283bbf67bd196d9bfe9331a59fb

Optional Packages                                                        <mark>Version</mark>
---------------------------------------------------- ---------------------------
xr-8000-l2mcast                                                   <mark>24.1.2v1.0.0-1</mark>
xr-8000-mcast                                                     24.1.2v1.0.0-1
xr-8000-netflow                                                   24.1.2v1.0.0-1
xr-bgp                                                            24.1.2v1.0.0-1
xr-ipsla                                                          24.1.2v1.0.0-1
xr-is-is                                                          24.1.2v1.0.0-1
xr-k9sec                                                          24.1.2v1.0.0-1
xr-lldp                                                           24.1.2v1.0.0-1
xr-mcast                                                          24.1.2v1.0.0-1
xr-mpls-oam                                                       24.1.2v1.0.0-1
xr-netflow                                                        24.1.2v1.0.0-1
xr-ops-script-repo                                                24.1.2v1.0.0-1
xr-ospf                                                           24.1.2v1.0.0-1
xr-perf-meas                                                      24.1.2v1.0.0-1
xr-perfmgmt                                                       24.1.2v1.0.0-1
xr-track                                                          24.1.2v1.0.0-1
RP/0/RP0/CPU0:8202-32FH-M_27#
</code>
</pre>
</div>

The last step is to check FPD versions and see if some still need a manual upgrade or not. Status should be ‘CURRENT’ or ‘NOT REQ’.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh hw-module fpd
Mon Jul  1 12:00:51.011 UTC

Auto-upgrade:Enabled
Attribute codes: B golden, P protect, S secure, A Anti Theft aware
                                                                         FPD Versions
                                                                        ==============
Location   Card type             HWver FPD device       ATR <mark>Status</mark>   Running Programd  Reload Loc
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
0/PM1      PSU2KW-ACPI           0.0   PO-PrimMCU           NOT READY                         N/A
0/PM1      PSU2KW-ACPI           0.0   PO-SecMCU            CURRENT    1.08    1.08       NOT REQ
0/FB0      8202-32FH-M[FB]       1.0   IoFpga               CURRENT    1.10    1.10       NOT REQ
0/FB0      8202-32FH-M[FB]       1.0   IoFpgaGolden     B   CURRENT            1.00       NOT REQ
</code>
</pre>
</div>

**Note:** It’s expected to see some FPD reporting ‘NEED UPGD’. Those are usually golden images used for disaster recovery. If you try to upgrade them, the system will prompt following log: fpd_client[318]: %PLATFORM-FPD_CLIENT-1-UPGRADE_SKIPPED : FPD upgrade skipped for BiosGolden@0/RP0/CPU0: Image not upgradable. It’s possible to force golden FPD upgrade using the ‘force’ keyword if advised by Cisco TAC.
{: .notice--info}


# install replace in action

The whole process can be observed in the 6min [video clip below]({{site.baseurl}}/images/8000%20install%20replace.gif):
![8000 install replace.gif]({{site.baseurl}}/images/8000%20install%20replace.gif)


# Conclusion
A Cisco IOS XR PhD is not required anymore to perform a software upgrade. Both install replace and Golden ISO can be leveraged to ease the process of staging, patching or upgrading the software running on Cisco 8000 routers. 

