---
published: false
date: '2024-09-02 15:48 +0200'
title: Cisco 8000 Disk Encryption
author: Frederic Cuiller
excerpt: >-
  This article aims to describe the disk encryption feature on Cisco 8000
  routers.  This functionality creates an encrypted partition where sensitive
  data such configuration, certificates and keys are stored. 
tags:
  - Cisco 8000
  - Disk Encryption
  - dm-crypt
  - TAm
position: hidden
---
# Introduction
This article aims to describe the disk encryption feature on Cisco 8000 routers.  
This functionality creates an encrypted partition where sensitive data such configuration, certificates and keys are stored. This secured partition is only accessible using a specific key which is stored inside Cisco 8000 Trust Anchor module (TAm) chip.

# How Does it Work?
Based on a Linux kernel, Cisco IOS XR leverages Linux dm-crypt infrastructure and additional components such Logical Volume Manager (LVM), Device-Mapper (DM), cryptsetup, Linux Unified Key Setup (LUKS); to transparently encrypt and decrypt data stored on the hard disk.

![8000-disk-encryption.png]({{site.baseurl}}/images/8000-disk-encryption.png)

The feature provides 150MB of secured storage which Cisco IOS XR applications can use at <code>/var/xr/enc</code>

# Disk Encryption Configuration

By default, disk encryption feature is disabled. This can be confirmed with following output:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#sh disk-encryption status
Fri Aug 16 23:54:26.500 UTC
Node:     node0_RP0_CPU0
--------------------------------------------------------------------------------
Mountpoint             Enc status             Mnt status             Prep status
--------------------------------------------------------------------------------
/var/xr/enc            Not Encrypted          Not Mounted            NA
</code>
</pre>
</div>

Additional, there is no secured partition mounted:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M#sh media
Mon Aug 26 13:47:23.471 UTC
Media Info for Location: node0_RP0_CPU0
Partition                            Size     Used  Percent    Avail
--------------------------------------------------------------------
rootfs:                             46.2G    19.1G      41%    27.1G
data:                               65.6G    12.8G      19%    52.9G
disk0:                               3.3G     4.5M       1%     3.1G
/var/lib/docker                      5.6G     720K       1%     5.3G
harddisk:                             51G      11G      22%      38G
log:                                 4.5G     491M      12%     3.7G
</code>
</pre>
</div>

Disk encryption can be enabled or disabled on-demand. Each change requires a system reload.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#disk-encryption ?
  activate    Activate disk encryption
  deactivate  Deactivate disk encryption
</code>
</pre>
</div>

**Note:** Once CLI is entered, the secured partition will be created and the system automatically reloads. System does not prompt for confirmation.
{: .notice--info}

On Cisco 8100/8200/8700 fixed systems, <code>disk-encryption activate</code> must be executed in XR exec mode:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8202-32FH-M_27#disk-encryption activate ?
  location  Specify location
RP/0/RP0/CPU0:8202-32FH-M_27#disk-encryption activate
Fri Aug 16 23:54:37.766 UTC
<mark>RP/0/RP0/CPU0:Aug 16 23:54:37.786 UTC: ssd_enc_server[271]: %OS-SSD_ENC-1-PREPARE_STARTED : Encryption activation prepare is in progress. Once it completes successfully, system will reload. Check encryption status by executing 'show disk-encryption status' CLI</mark>
Preparing system for backup. This may take a few minutes especially for large configurations.
	Status report: node0_RP0_CPU0: BACKUP INPROGRESS
RP/0/RP0/CPU0:8202-32FH-M_27#	Status report: node0_RP0_CPU0: BACKUP HAS COMPLETED SUCCESSFULLY
[Done]

[  OK  ] Stopped target XR installation and startup.
[  OK  ] Stopped target Multi-User System.
[  OK  ] Stopped target Login Prompts.
[  OK  ] Stopped target Containers.
snip
<mark>RP/0/RP0/CPU0:Aug 16 23:54:47.206 UTC: shelfmgr[236]: %PLATFORM-SHELFMGR-4-CARD_RELOAD : Reloading 0/RP0/CPU0: SSD encryption initiated card reload</mark>
[  OK  ] Stopped Permit User Sessions.
[  OK  ] Stopped target Remote File Systems.
Snip
[4017853.656053] pm-fpga: Turning Status LED to amber, Active, Alarm, Board and Chassis Attention LED to OFF …
[4017853.681085] pm-fpga: Power cycling local CPU power zone
</code>
</pre>
</div>

During reload phase, LVM cryptsetup and LUKS can be seen in action:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Cisco 8000(R) Series BIOS Ver 1.09 Primary
Intel(R) Xeon(R) CPU D-1530 @ 2.40GHz
Board Type 0x111 PID 8202-32FH-M Serial FLM260902HU
X86FPGA 1.7.0 TamLib 5.01.01
Booting from Disk
  Booting `IOS-XR-latest'

Booting latest from Disk..
Loading Kernel..
snip
fstab modified mount options for 1 partitions
Setup encrypted LV...
Execute: cryptsetup luksOpen /dev/main-xr-vg/install-data-encrypted encrypted -d '-'
snip
[  OK  ] Found device Micron_5300_MTFDDAV480TDS partition01.
[  OK  ] Found device Micron_5300_MTFDDAV480TDS DR-Part.
[  OK  ] Found device /dev/mapper/encrypted.
[  OK  ] Found device /dev/main-xr-vg/install-data-scratch.
[  OK  ] Found device /dev/main-xr-vg/install-data-disk1.
[  OK  ] Found device /dev/main-xr-vg/install-data-docker.
[  OK  ] Found device /dev/main-xr-vg/install-data-log.
[  OK  ] Found device /dev/main-xr-vg/boot-lv.
[  OK  ] Created slice system-lvm2\x2dpvscan.slice.
         Starting LVM event activation on device 8:3...
         Starting File System Check on /dev/main-xr-vg/boot-lv...
         Starting File System Check…in-xr-vg/install-data-disk1...
         Starting File System Check…n-xr-vg/install-data-docker...
         Starting File System Check…main-xr-vg/install-data-log...
         Starting File System Check…-xr-vg/install-data-scratch...
         Starting File System Check on /dev/mapper/encrypted...
         Starting File System Check on /dev/sda1...
         Starting File System Check on /dev/sda2...
[  OK  ] Started LVM event activation on device 8:3.
[  OK  ] Started File System Check on /dev/sda1.
[  OK  ] Started File System Check on /dev/sda2.
         Mounting /mnt/dr_part...
[  OK  ] Started File System Check on /dev/mapper/encrypted.
         Mounting /var/xr/enc...
[  OK  ] Started File System Check on /dev/main-xr-vg/boot-lv.
         Mounting /boot...
[  OK  ] Started File System Check …in-xr-vg/install-data-scratch.
[  OK  ] Started File System Check …ain-xr-vg/install-data-docker.
         Mounting /var/lib/docker...
         Mounting /var/xr/scratch...
[  OK  ] Mounted /mnt/dr_part.
<mark>[  OK  ] Mounted /var/xr/enc.<mark>
[  OK  ] Mounted /boot.
         Mounting /boot/efi...
[  OK  ] Mounted /var/lib/docker.
[  OK  ] Mounted /var/xr/scratch.
[  OK  ] Mounted /boot/efi.
snip
SYSTEM CONFIGURATION COMPLETED
</code>
</pre>
</div>


# Conclusion
