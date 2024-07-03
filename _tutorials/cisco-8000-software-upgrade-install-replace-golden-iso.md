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

**Note:** In case of a software upgrade (or downgrade, the system is still reloaded to boot on the target software release.
{: .notice--info}

# Prerequisite

Before upgrading, the system must be in SIA compliant state. Software Innovation Access (SIA) subscription is required to access to the latest Feature Releases. SMU installation is still possible for a given release if the system is non-compliant.  

The following command allow to check compliancy:

RP/0/RP0/CPU0:Saturn-8711-32FH-M#sh license platform summary
Tue Jul  2 13:48:07.272 UTC
Collection: LAST: Tue Jul 02 2024 13:13:37 UTC
            NEXT: Tue Jul 02 2024 14:13:37 UTC
Reporting:  LAST: Tue Jul 02 2024 09:13:37 UTC
            NEXT: Wed Jul 03 2024 09:13:37 UTC
SIA Status: In Compliance
                                                                   Count
Feature/Area     Entitlement                                     Last Next
============ =================================================== ==== ====
FCM 2 Core/E Core and Edge Premier RTU per 100G for Cisco 8000E     4    4
FCM 2 Core/E 8000 and 8000E Type C Device SIA FCM 2.0               2    2

If the system is not compliant, install replace procedure will fail as illustrated below:

RP/0/RP0/CPU0:8202-32FH-M_27#show license platform detail
Mon Jul  1 11:31:03.513 UTC
Collection: LAST: Mon Jul 01 2024 11:09:05 UTC
            NEXT: Mon Jul 01 2024 12:09:05 UTC
Reporting:  LAST: Sun Jun 30 2024 19:09:05 UTC
            NEXT: Mon Jul 01 2024 19:09:05 UTC
**********************************IMPORTANT************************************
SIA Status: Out of Compliance(Grace Period Expired)
            SW Upgrades are blocked as SIA grace period has expired
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
RP/0/RP0/CPU0:Jul  1 11:30:17.193 UTC: smart_lic_sia[65989]: %LICENSE-PLAT_CLIENT-2-SIA_EVAL_EXPIRED : SW Upgrades are blocked as SIA grace period has expired
.RP/0/RP0/CPU0:Jul  1 11:30:19.968 UTC: instorch[147]: %INFRA-INSTALL-3-VETO : Install operation has been vetoed by 1 install clients. First veto is from 0/RP0/CPU0 on plat_sl_client in process 369: CHECK: Unable to proceed. Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai
RP/0/RP0/CPU0:Jul  1 11:30:19.980 UTC: instorch[147]: %INFRA-INSTALL-3-ACTION_FAILED : Packaging operation 1.1.1 failed - Operation vetoed by 1 install client. Error is from plat_sl_client on 0/RP0/CPU0: CHECK: Unable to proceed. Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai
RP/0/RP0/CPU0:Jul  1 11:30:19.980 UTC: instorch[147]: %INFRA-INSTALL-3-ACTION_FAILED : Install operation (Packaging operation) 1.1.1 failed - Operation vetoed by 1 install client. Error is from plat_sl_client on 0/RP0/CPU0: CHECK: Unable to proceed. Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai
.....^@.

Atomic change 1.1: 'install replace /harddisk:/8000-golden-x86_64-24.1.2-1337.iso' failed - Operation vetoed by 1 install client. Error is from plat_sl_client on 0/RP0/CPU0: CHECK: Unable to proceed.
Device upgrade is not allowed due to SIA Out of Compliance, please check 'show license platform detai. Query 'show install history id 1.1.1 errors location 0/RP0/CPU0' for more details and next steps. Automatically recovered after failure, ready for next user request.
RP/0/RP0/CPU0:8202-32FH-M_27#



