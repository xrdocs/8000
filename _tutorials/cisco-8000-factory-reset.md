---
published: true
date: '2024-06-26 15:20 +0200'
title: Cisco 8000 Factory Reset
author: Frederic Cuiller
tags:
  - iosxr
  - Cisco 8000
  - factory reset
position: top
excerpt: >-
  This article introduces factory reset functionnality which safely erases any
  sensitive data stored on the system.
---
{% include toc icon="table" title="Cisco 8000 Factory Reset" %}

# Introduction

Factory reset functionality was released in IOS XR 7.3.4. This feature aims to erase and overwrite any sensitive data, configuration, or keys present in the Cisco 8000 route processor or line card. It ensures media sanitization and prevents unauthorized data retrieval from 3rd parties.  

It removes following folders on RP and line cards:

- /misc/disk1
- /misc/scratch
- /var/log
- /misc/config

The whole procedure takes approximately 15min.  

Factory reset feature complies with NIST SP 800-88 guidelines for Media Sanitization. This is achieved by removing all files from the partition and then fill it up with random data using dd utility.

# Factory Reset steps

The system used for this demo is a Cisco 8200 fixed system running IOS XR 24.2.1.
It already contains configuration and a couple of user’s files are present on the harddisk (ISO, configuration backup, packet capture, etc.):

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#<mark>sh run</mark>
Mon Jun 24 20:14:50.522 UTC
!! Building configuration...
!! IOS XR Configuration 24.2.1
!! Last configuration change at Mon Jun 24 20:10:24 2024 by cisco
!
hostname 8212-48FH-M_P4C-006
username cisco
 group root-lr
 group cisco-support
 secret 10 $6$93Y4r//dlROQ5r/.$UZTVZ37hLffCn53ElnuTeF.apGudFuFJ7.VrllETStXu/J.dv6Jcr4da9N.YZDt8yNpkYGIBuXmDxxKY4FCyu1
!
line console
 exec-timeout 0 0
!
arp vrf default 10.10.0.2 0001.0000.0000 ARPA
arp vrf default 10.10.1.2 0001.0000.0001 ARPA
arp vrf default 10.10.2.2 0001.0000.0002 ARPA
arp vrf default 10.10.3.2 0001.0000.0003 ARPA
arp vrf default 10.10.4.2 0001.0000.0004 ARPA
arp vrf default 10.10.5.2 0001.0000.0005 ARPA
arp vrf default 10.10.6.2 0001.0000.0006 ARPA
arp vrf default 10.10.7.2 0001.0000.0007 ARPA
arp vrf default 10.10.8.2 0001.0000.0008 ARPA
arp vrf default 10.10.9.2 0001.0000.0009 ARPA
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#dir harddisk:
Mon Jun 24 20:15:01.818 UTC

Directory of harddisk:
 655361 drwxrwxrwx. 2       4096 Feb  7 19:40 custom_metrics
     34 -rw-rw-rw-. 1       4709 Mar 19 00:02 show_drops.txt
     30 -rwxrwxrwx. 1   17621540 Mar 12 18:25 sfms-1-9-g966e7d7-s.bin.signed.cap
     23 -rw-r--r--. 1   26609522 May 11 04:04 npu_drvr_4793.by.6.20240511-040103.node0_RP0_CPU0.71096.core.gz
     15 -rw-rw-rw-. 1      22948 Feb  9 19:16 slugger_2.cfg
     46 -r-xr-xr-x. 1 1698416640 Apr  8 21:31 8000-x64.iso_slugger_412
     40 -rw-r--r--. 1      28845 May  2 19:59 npu_pstack.txt
2621441 drwxrwxrwx. 2       4096 Feb  7 19:42 dumper
 131073 drwxrwxrwx. 3       4096 Feb  7 19:40 mirror
     31 -rw-r--r--. 1     379524 May 31 18:08 npu_drvr_5014.by.11.20240531-180744.node0_RP0_CPU0.5a1e7.core.txt
1835009 drwxrwxrwx. 2       4096 Feb  7 19:41 .sppdc_new
     11 drwx------. 2      16384 Feb  7 19:29 lost+found
     21 -rw-r--r--. 1  492187234 Feb 21 19:34 <mark>8000-x86_64-24.2.1.23I-CSCwj06962-deenayak-2024-02-20-1935.tgz</mark>
 786433 drwxrwxrwx. 3       4096 Feb  7 19:45 var
 393217 drwxrwxrwx. 2      20480 Jun 14 16:43 showtech
     35 -rw-rw-rw-. 1          0 Mar 19 00:02 show_drops_error.txt
 917505 drwxrwxrwx. 2       4096 Jun 13 18:08 npdatalog
     39 -rw-rw-rw-. 1      24576 Apr  2 00:05 ptp_trace_file.log
1179649 drwxrwxrwx. 2       4096 Feb 21 19:25 dph_mon
2490369 drwxr-xr-x. 2       4096 Jun 24 19:10 nvgen_traces
     13 -rw-rw-rw-. 1      20862 Jun 24 18:14 debug_shell_client.log
3014657 drwxr-xr-x. 3       4096 Feb  7 19:42 pam
2883585 drwxr-xr-x. 2       4096 Feb  7 19:34 nvram
1048577 drwxrwxrwx. 3       4096 Feb  7 19:40 ztp
1703937 drwxr-xr-x. 3       4096 Feb 21 00:15 tmp_archive
 524289 drwxrwxrwx. 2      12288 Jun 24 19:17 .sppdc
     28 -rw-r--r--. 1  191145143 May 31 18:08 npu_drvr_5014.by.11.20240531-180744.node0_RP0_CPU0.5a1e7.core.gz
3145729 drwxrwxrwx. 2       4096 Jun 24 18:14 npu_sdk_logs
     29 -rw-rw-rw-. 1      32290 Mar  8 23:02 slugger_1_bo_macsec.txt
     20 -r-xr-xr-x. 1 1839591167 Feb 20 23:40 <mark>8000-x64-usb-24.2.1.23I.zip</mark>
     26 -rw-r--r--. 1      33964 May 16 02:32 tmgctrl_core_show_ctrl_2224.by.6.20240516-023216.node0_RP0_CPU0.d741a.core.txt
     41 -rw-rw-rw-. 1         65 Apr 12 16:48 utc_offset.json
2097153 drwxr-xr-x. 2       4096 May 21 22:19 smu
     17 -rw-r--r--. 1   14463964 Jun 13 18:13 var_log_pre_reimage.tgz
     14 -rw-rw-rw-. 1         32 Jun 24 18:15 auto-fpd-upgrade-config.bin
     22 -rw-rw-rw-. 1      22769 Feb 21 23:32 <mark>slugger_2.conf</mark>
     45 -rw-rw-rw-. 1       3431 Jun 13 17:44 fcm_data
 262145 drwxrwxrwx. 5       4096 Feb  9 19:17 shutdown
1310721 drwxrwxrwx. 5       4096 Jun 24 18:20 cisco_support
     18 -rwxr-xr-x. 1    3526464 Feb 15 00:23 eeupdate64e
     16 -rw-rw-rw-. 1       5392 Feb  9 18:46 feature_list
2359297 drwx------. 3       4096 Jun 24 18:16 ima
     24 -rw-r--r--. 1     380534 May 11 04:04 npu_drvr_4793.by.6.20240511-040103.node0_RP0_CPU0.71096.core.txt
     25 -rw-r--r--. 1    5479605 May 16 02:32 tmgctrl_core_show_ctrl_2224.by.6.20240516-023216.node0_RP0_CPU0.d741a.core.gz
     19 -rwxr-xr-x. 1    1048576 Feb 15 00:24 Cisco_I210_Sgmii_MDIO_NCSI_8Mb_A2_8000081A.bin
     12 -rw-rw-rw-. 1   10486742 Jun 24 20:14 nvgen_bkup.log

53330272 kbytes total (44088536 kbytes free)
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#
</code>
</pre>
</div>

Factory reset procedure is invoked with “factory-reset” command. It contains a couple of options:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#factory-reset ?
  reload    Reload the location after performing factory-reset
  shutdown  Shutdown the location after performing factory-reset
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#factory-reset reload ?
  location  Specify location
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#factory-reset reload location ?
  0/RP0/CPU0  Fully qualified location specification
  WORD        Fully qualified location specification
  all         Show all locations
</code>
</pre>
</div>

**Note:** In case of Cisco 8800 distributed systems, it’s possible to wipe a specific line card or the standby RP for instead. Another option is to shutdown the system once the reset is done. This can help field engineers identifying the unit which must be replaced in case of RMA.
{: .notice--primary}

Once the procedure is launched:

- Encryption keys are first deleted
- shelfmgr process reloads the system/line card
- IOS XR processes are stopped
- Full disk erasure starts

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#<mark>factory-reset reload location 0/RP0/CPU0</mark>
Mon Jun 24 20:18:32.935 UTC
Performing factory-reset may affect the stability of the system. Re-imaging maybe required to recover. Continue?
[confirm]
<mark>RP/0/RP0/CPU0:Jun 24 20:18:41.523 UTC: ssd_enc_server[385]: %OS-SSD_ENC-1-FACTORY_RESET : Factory reset: Deleted SSD encryption keys successfully</mark>
	Status report: node0_RP0_CPU0: START TO BACKUP

Preparing system for backup. This may take a few minutes especially for large configurations.
RP/0/RP0/CPU0:8212-48FH-M_P4C-006#	Status report: node0_RP0_CPU0: BACKUP HAS COMPLETED SUCCESSFULLY
[Done]
[  OK  ] Stopped target XR installation and startup.
[  OK  ] Stopped target Multi-User System.
[  OK  ] Stopped target Login Prompts.
[  OK  ] Stopped target Containers.
snip
[  OK  ] Stopped Marvell Switch Setup.
[  OK  ] Stopped Load/Save Random Seed.
[  OK  ] Removed slice system-lvm2\x2dpvscan.slice.
[  OK  ] Removed slice system-serial\x2dgetty.slice.
<mark>RP/0/RP0/CPU0:Jun 24 20:18:46.421 UTC: shelfmgr[444]: %PLATFORM-SHELFMGR-4-CARD_RELOAD : Reloading 0/RP0/CPU0: Factory reset initiated card reload</mark>
         Stopping Permit User Sessions...
[  OK  ] Stopped Redis In-Memory Data Store.
snip
[  OK  ] Stopped IOS-XR ISO Installation including sshd.
         Stopping Service for factory reset...
[  OK  ] Stopped OpenSSH Key Generation.
<mark>[ 7519.819442] factory_reset_stop.sh[39073]: Mon Jun 24 20:18:55 UTC 2024 Factory reset requested</mark>
</code>
</pre>
</div>

- watchdog is temporarily disabled to let enough time to complete the procedure. It prevents system from reloading automatically after some time.
- Cleanup starts

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>[ 7521.654435] factory_reset_stop.sh[39073]: Mon Jun 24 20:18:57 UTC 2024 Started punching watchdog</mark>
[ 7522.762247] factory_reset_stop.sh[39073]: Mon Jun 24 20:18:58 UTC 2024 <mark>Started cleaning up mount point: /misc/scratch</mark>
[ 7550.347910] factory_reset_stop.sh[39073]: Mon Jun 24 20:19:26 UTC 2024 Started syncing folder: /misc/scratch
[ 7556.500349] factory_reset_stop.sh[39073]: Mon Jun 24 20:19:32 UTC 2024 Finished syncing folder: /misc/scratch
[ 7557.504424] factory_reset_stop.sh[39073]: Mon Jun 24 20:19:33 UTC 2024 Finished cleaning up mount point: /misc/scratch
[ 7558.508476] factory_reset_stop.sh[39073]: Mon Jun 24 20:19:34 UTC 2024 +++++++++++++++++++++++++++++++++++++++++++++++
[ 7559.512463] factory_reset_stop.sh[39073]: Mon Jun 24 20:19:35 UTC 2024 Started cleaning up mount point: /var/log
[ 7605.168217] factory_reset_stop.sh[39073]: Mon Jun 24 20:20:20 UTC 2024 Started syncing folder: /var/log
[ 7614.630737] factory_reset_stop.sh[39073]: Mon Jun 24 20:20:30 UTC 2024 Finished syncing folder: /var/log
[ 7615.634751] factory_reset_stop.sh[39073]: Mon Jun 24 20:20:31 UTC 2024 Finished cleaning up mount point: /var/log
[ 7616.638846] factory_reset_stop.sh[39073]: Mon Jun 24 20:20:32 UTC 2024 +++++++++++++++++++++++++++++++++++++++++++++++
[ 7617.642835] factory_reset_stop.sh[39073]: Mon Jun 24 20:20:33 UTC 2024 Started cleaning up mount point: /misc/disk1
[ 8049.715296] factory_reset_stop.sh[39073]: Mon Jun 24 20:27:45 UTC 2024 Started syncing folder: /misc/disk1
[ 8079.987454] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:15 UTC 2024 Finished syncing folder: /misc/disk1
[ 8080.991509] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:16 UTC 2024 Finished cleaning up mount point: /misc/disk1
[ 8081.995598] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:17 UTC 2024 +++++++++++++++++++++++++++++++++++++++++++++++
[ 8082.999767] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:18 UTC 2024 Started cleaning up folder: /misc/config
[ 8083.022996] factory_reset_stop.sh[39073]: /etc/init.d/factory_reset_stop.sh: line 181: warning: command substitution: ignored null byte in input
[ 8085.173683] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:20 UTC 2024 Started syncing folder: /misc/config
[ 8086.187693] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:21 UTC 2024 Finished syncing folder: /misc/config
[ 8087.191848] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:22 UTC 2024 Finished cleaning up folder: /misc/config
[ 8088.195930] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:23 UTC 2024 +++++++++++++++++++++++++++++++++++++++++++++++
[ 8089.199929] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:24 UTC 2024 Started cleaning up folder: /var/xr/enc/misc/config
[ 8090.203901] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:25 UTC 2024 /var/xr/enc/misc/config not present
[ 8091.207959] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:26 UTC 2024 Finished cleaning up folder: /var/xr/enc/misc/config
[ 8092.211971] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:27 UTC 2024 +++++++++++++++++++++++++++++++++++++++++++++++
[ 8093.215952] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:28 UTC 2024 Started cleaning up folder: /mnt/rootfs/misc/config
[ 8094.219960] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:29 UTC 2024 /mnt/rootfs/misc/config not present
[ 8095.224131] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:30 UTC 2024 <mark>Finished cleaning up folder: /mnt/rootfs/misc/config</mark>
[ 8096.228311] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:31 UTC 2024 +++++++++++++++++++++++++++++++++++++++++++++++
[ 8097.233317] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:32 UTC 2024 Encrypted logical volume does not exist. Nothing to remove.
</code>
</pre>
</div>

- watchdog is restored and system reloads (or performs shutdown depending option used)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
[ 8099.290130] factory_reset_stop.sh[39073]: 2024-06-24 20:28:34.972 UTC: <mark>Stopped punching watchdog</mark>
[ 8100.347121] factory_reset_stop.sh[39073]: Mon Jun 24 20:28:36 UTC 2024 Stopped punching watchdog
snip
[  OK  ] Stopped Monitoring of LVM2… dmeventd or progress polling.
[  OK  ] Reached target Shutdown.
[  OK  ] Reached target Final Step.
<mark>[  OK  ] Started Reboot.</mark>
<mark>[  OK  ] Reached target Reboot.</mark>
[ 8102.987301] systemd-shutdown[1]: Could not detach DM /dev/dm-9: Device or resource busy
[ 8102.997085] systemd-shutdown[1]: Could not detach DM /dev/dm-8: Device or resource busy
[ 8103.006835] systemd-shutdown[1]: Could not detach DM /dev/dm-7: Device or resource busy
[ 8103.038574] systemd-shutdown[1]: Failed to finalize  DM devices, ignoring
[ 8103.414279] pm-fpga: Turning Status LED to amber, Active, Alarm, Board and Chassis Attention LED to OFF ...
<mark>[ 8103.436109] pm-fpga: Power cycling local CPU power zone</mark>
</code>
</pre>
</div>

In this demo, system performs a fresh boot like it was out of factory

<div class="highlighter-rouge">
<pre class="highlight">
<code>
Cisco 8000(R) Series BIOS Ver 1.09 Primary
Intel(R) Xeon(R) CPU D-1633N @ 2.50GHz
Board Type 0x10D PID 8212-48FH-M Serial FLM274203G9
X86FPGA 1.7.0 TamLib 5.01.01
Booting from Disk
  Booting `IOS-XR-latest'

<mark>Booting latest from Disk..</mark>
Loading Kernel..
Verifying /bzImage...
/bzImage verified using attached signature.
Verifying /bzImage...
/bzImage verified using attached signature.
Loading initrd..
Verifying /initrd.img...
/initrd.img verified using Pkcs7 signature.
snip
[   58.938818] xrnginstall[1970]: 2024-06-24 20:30:48.532 UTC: xrinstall: XR Lineup:  r242x.lu%EFR-00000459299
[   58.941807] xrnginstall[1974]: 2024-06-24 20:30:48.535 UTC: xrinstall: XR Version: 24.2.1
[   58.944949] xrnginstall[1978]: 2024-06-24 20:30:48.538 UTC: xrinstall: Completed set up of dumper and build info files
[   58.948153] xrnginstall[1982]: 2024-06-24 20:30:48.541 UTC: xrinstall: Preparing IOS-XR
[   58.951203] xrnginstall[1986]: 2024-06-24 20:30:48.544 UTC: xrinstall: Already pointing at latest FS
[   59.539218] xrnginstall[2321]: 2024-06-24 20:30:49.132 UTC: xrinstall: Load platform kernel modules
[   59.650429] xrnginstall[2373]: 2024-06-24 20:30:49.244 UTC: xrinstall: Succeeded loading platform kernel modules
[   59.655352] xrnginstall[2378]: 2024-06-24 20:30:49.249 UTC: xrinstall: Enable hardware
[   60.518923] xrnginstall[2503]: 2024-06-24 20:30:50.112 UTC: xrinstall: Succeeded enabling hardware
[   60.747820] xrnginstall[2514]: 2024-06-24 20:30:50.341 UTC: xrinstall: Creating owner and partner internal repositories
[   60.751067] xrnginstall[2518]: 2024-06-24 20:30:50.344 UTC: xrinstall: Setting up owner and partner RPM symlinks
[   60.762977] xrnginstall[2526]: 2024-06-24 20:30:50.356 UTC: xrinstall: Setting up owner/partner package list symlink
[   60.764483] xrnginstall[2528]: ln: failed to create symbolic link '/var/www/owner_partner_package_list': No such file or directory
[   60.767584] xrnginstall[2532]: 2024-06-24 20:30:50.361 UTC: xrinstall: Installing partner keys if required
[   60.785359] xrnginstall[2545]: 2024-06-24 20:30:50.379 UTC: xrinstall: Checking if rollback cleanup is required
[   60.789294] xrnginstall[2550]: 2024-06-24 20:30:50.383 UTC: xrinstall: Finished rollback cleanup stage
[   60.795784] xrnginstall[2556]: 2024-06-24 20:30:50.389 UTC: xrinstall: Single node: starting XR
[   60.929697] xrnginstall[2648]: 2024-06-24 20:30:50.523 UTC: xrinstall: xrnginstall completed successfully
<mark>ios con0/RP0/CPU0 is now available</mark>
<mark>Press RETURN to get started.</mark>

RP/0/RP0/CPU0:Jun 24 20:31:13.204 UTC: ssd_enc_server[385]: %OS-SSD_ENC-1-ENCRYPTED_LV_RECOVERED : Encrypted logical volume failure seen during boot. Recovered by recreating LV and encryption keys. Check /misc/disk1 for error logs.

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

RP/0/RP0/CPU0:Jun 24 20:32:43.041 UTC: pyztp2[125]: %INFRA-ZTP-4-START : ZTP has started. Interfaces might be brought up if they are shutdown
</code>
</pre>
</div>

Like any fresh install, the first thing to do is to configure a local administrator account. Once logged in, we can see there is no configuration anymore and the files previously located on the harddisk are no longer present

<div class="highlighter-rouge">
<pre class="highlight">
<code>
!!!!!!!!!!!!!!!!!!!! NO root-system username is configured. Need to configure root-system username. !!!!!!!!!!!!!!!!!!!!

         --- Administrative User Dialog ---


  Enter root-system username:
  % Entry must not be null.

  Enter root-system username: cisco
  Enter secret:
  Enter secret again:
Use the 'configure' command to modify this configuration.
User Access Verification

Username: cisco
Password:

RP/0/RP0/CPU0:ios#
RP/0/RP0/CPU0:ios#
RP/0/RP0/CPU0:ios#
RP/0/RP0/CPU0:ios#<mark>sh run</mark>
Mon Jun 24 20:33:14.406 UTC
!! Building configuration...
!! IOS XR Configuration 24.2.1
!! Last configuration change at Mon Jun 24 20:32:59 2024 by SYSTEM
!
username cisco
 group root-lr
 group cisco-support
 secret 10 $6$NJN4//JaFEN22//.$r.F4SlhQTmLYUJnhfJpeYTpuNqn0ras7vp/fx7e0Bf3MkqPKz8Yj8su2SJVj7KOOmEYW161.LM.gFz2fa65OA0
!
interface MgmtEth0/RP0/CPU0/0
 shutdown
!
-	snip -
interface FourHundredGigE0/0/0/47
 shutdown
!
end

RP/0/RP0/CPU0:ios#<mark>dir harddisk:</mark>
Mon Jun 24 20:33:21.665 UTC

Directory of harddisk:
     32 -rw-r--r--. 1  8250 Jun 24 20:28 factory_reset.txt
 131073 drwxrwxrwx. 3  4096 Jun 24 20:32 mirror
1835009 drwxrwxrwx. 2  4096 Jun 24 20:32 .sppdc_new
     11 drwx------. 2  4096 Jun 24 20:30 lost+found
 393217 drwxrwxrwx. 2  4096 Jun 24 20:32 showtech
 786433 drwxrwxrwx. 2  4096 Jun 24 20:32 dph_mon
     14 -rw-rw-rw-. 1    60 Jun 24 20:31 debug_shell_client.log
2883585 drwxrwxrwx. 2  4096 Jun 24 20:30 nvram
1048577 drwxrwxrwx. 3  4096 Jun 24 20:32 ztp
 524289 drwxrwxrwx. 2  4096 Jun 24 20:32 .sppdc
3145729 drwxrwxrwx. 2  4096 Jun 24 20:31 npu_sdk_logs
     15 -rw-rw-rw-. 1    32 Jun 24 20:31 auto-fpd-upgrade-config.bin
     12 -rw-r--r--. 1 22012 Jun 24 20:30 enc_error_20240624_203018.log
 262145 drwxrwxrwx. 2  4096 Jun 24 20:30 shutdown
2359297 drwx------. 3  4096 Jun 24 20:32 ima
     13 -rw-rw-rw-. 1 59716 Jun 24 20:33 nvgen_bkup.log

53330272 kbytes total (50538020 kbytes free)
</code>
</pre>
</div>

# Factory Reset in Action

The whole process can be observed in this 5min video clip
![8000 factory reset.gif]({{site.baseurl}}/images/8000 factory reset.gif)


# Conclusion

Cisco 8000 routers can now be safely erased before being returned to the depot as part of RMA procedure using factory reset procedure. The system will keep running the same IOS XR release (version, packages, SMU) but is purged from any sensitive data.
