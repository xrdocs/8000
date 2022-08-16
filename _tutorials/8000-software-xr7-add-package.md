---
published: true
date: '2022-08-16 14:50 +0200'
title: 'Cisco 8000 software manageability: how to add a package'
author: Frederic Cuiller
position: top
excerpt: >-
  This blog post aims to describe IOS XR7 software operation details to add
  optional packages on a Cisco 8000 router.
tags:
  - iosxr
  - XR7
  - Cisco 8000
  - install
---
{% include toc icon="table" title="Cisco 8000 software manageability: how to add a package" %}

## Introduction 

This blog post aims to describe IOS XR7 software operation details to add optional packages on a Cisco 8000 router.

## History 

Cisco IOS-XR is a modular Network Operating System created in 2004 for CRS. It was originally built using a 32-bit QNX based micro kernel architecture. It then evolved toward a 64-bit Linux-based kernel with IOS-XR 6 introduced with NCS 5500. Those 2 versions are referenced as classic IOS-XR (cXR) and IOS-XR 64bit (or enhanced IOS-XR or eXR).  

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
k9 package is still required for crypto dataplane features such MACsec.
{: .notice--primary}

## Adding optional packages
After USB or PXE boot, network operator might need to add optional packages to implement features not covered in the Core Bundle (or mini.iso). telnet package for IOS-XR 7.3.15 will be used as an example.
   
After downloading and extracting 8000-optional-rpms.7.3.15.tar file from CCO, individual packages files can be found in their respective folders: 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
{25/08/21 14:51}dhcp-10-61-108-172:~/Downloads/IOS/8000 fcuiller% tree optional-rpms/
optional-rpms
├── cdp
│   ├── xr-cdp-3692251fac396a2d-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-78bb1e1f26c2908f-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8101-32h-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8102-64h-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8201-32fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8201-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8202-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8608-rp1-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-88-lc0-34h14fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-88-lc0-36fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-88-lc0-36fh-m-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8800-lc-36fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8800-lc-48h-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-cdp-8800-rp-7.3.15v1.0.0-1.x86_64.rpm
│   └── xr-cdp-d17f630e9aaec8a3-7.3.15v1.0.0-1.x86_64.rpm
├── healthcheck
│   ├── xr-healthcheck-3692251fac396a2d-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8101-32h-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8102-64h-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8201-32fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8201-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8202-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8608-rp1-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-88-lc0-34h14fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-88-lc0-36fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-88-lc0-36fh-m-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8800-lc-36fh-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8800-lc-48h-7.3.15v1.0.0-1.x86_64.rpm
│   ├── xr-healthcheck-8800-rp-7.3.15v1.0.0-1.x86_64.rpm
│   └── xr-healthcheck-d17f630e9aaec8a3-7.3.15v1.0.0-1.x86_64.rpm
<mark>└── telnet
    ├── xr-telnet-3692251fac396a2d-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8101-32h-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8102-64h-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8201-32fh-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8201-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8202-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8608-rp1-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-88-lc0-34h14fh-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-88-lc0-36fh-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-88-lc0-36fh-m-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8800-lc-36fh-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8800-lc-48h-7.3.15v1.0.0-1.x86_64.rpm
    ├── xr-telnet-8800-rp-7.3.15v1.0.0-1.x86_64.rpm
    └── xr-telnet-d17f630e9aaec8a3-7.3.15v1.0.0-1.x86_64.rpm</mark>

3 directories, 47 files
</code>
</pre>
</div>

Several files can be found:
- *xr-telnet-7.3.15v1.0.0-1.x86_64.rpm* is top-level package for telnet
- *xr-telnet-3692251fac396a2d-7.3.15v1.0.0-1.x86_64.rpm* and *xr-telnet-d17f630e9aaec8a3-7.3.15v1.0.0-1.x86_64.rpm* are partition packages for telnet
- all others are PID-specific packages for telnet

To install telnet specifically, a new TAR archive must be created with those files only:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
25/08/21 12:11}dhcp-10-61-108-172:~/Downloads/IOS/8000/optional-rpms fcuiller% <mark>tar cvf 8000-telnet-rpms.7.3.15.tar telnet/ </mark>
a telnet
a telnet/xr-telnet-88-lc0-36fh-m-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8608-rp1-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8800-lc-36fh-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8800-rp-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8202-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8102-64h-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-88-lc0-36fh-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8800-lc-48h-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-3692251fac396a2d-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8101-32h-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-88-lc0-34h14fh-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8201-32fh-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-d17f630e9aaec8a3-7.3.15v1.0.0-1.x86_64.rpm
a telnet/xr-telnet-8201-7.3.15v1.0.0-1.x86_64.rpm
</code>
</pre>
</div>

TAR archive must be transferred on the Cisco 8000 harddisk. SCP is used here:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#<mark>scp cisco@192.168.122.1:~/8000-telnet-rpms.7.3.15.tar /harddisk:/</mark>
Wed Aug 25 10:13:52.323 UTC
Connecting to 192.168.122.1...
Password:
  Transferred 235520 Bytes
  235520 bytes copied in 0 sec (6365405)bytes/sec

RP/0/RP0/CPU0:8201-32FH-1#dir harddisk:
Wed Aug 25 10:14:04.704 UTC

Directory of harddisk:
 655361 drwx------. 3   4096 Aug 25 09:49 ima
1441793 drwxr-xr-x. 2   4096 Aug 25 09:51 showtech
1310721 drwxrwxrwx. 2   4096 Aug 25 09:51 dumper
     13 -rw-rw-rw-. 1    127 Aug 25 09:54 feature_list
3932161 drwxrwxrwx. 2   4096 May 17 22:58 .sppdc_new
2883585 drwxrwxrwx. 3   4096 May 17 22:59 ztp
3145729 drwxr-xr-x. 3   4096 Aug 25 09:51 pam
 786433 drwxrwxrwx. 5   4096 Aug 25 09:51 cisco_support
 262145 drwxrwxrwx. 3   4096 May 17 22:59 mirror
     11 drwx------. 2  16384 May 17 22:51 lost+found
 524289 drwxrwxrwx. 2   4096 May 17 22:56 nvram
1048577 drwxrwxrwx. 2   4096 May 17 22:56 shutdown
<mark>     12 -rw-r--r--. 1 235520 Aug 25 10:13 8000-telnet-rpms.7.3.15.tar</mark>
1835009 drwxrwxrwx. 2   4096 Aug 25 09:46 npu_sdk_logs
 393217 drwxrwxrwx. 2   4096 Aug 25 09:47 .sppdc
</code>
</pre>
</div>

Installation can start.  

Current active software configuration doesn’t contain telnet package: 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active summary
Wed Aug 25 10:14:10.028 UTC
Active Packages:    XR: 178    All: 1274
Label:              7.3.15
Software Hash:      9149973e08793c3f44e84a4a5b385c8a

Optional Packages                                                        Version
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

First step is to add the package. This is accomplished with *install package add* command:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#<mark>install package add source ?</mark>
  WORD            Configured DNF repository name to use as the source
  WORD            Full path to a local directory to use as the source - must be a subdirectory of /var/xr/disk1/, /harddisk:/, or /misc/disk1/
<mark>  WORD            Full path to a local tar file to use as the source - must be located in or under /var/xr/disk1/, /harddisk:/, or /misc/disk1/</mark>
  any-configured  Use any configured repository to obtain packages
  ftp:            Remote repo ftp://server[;vrf]/remote_path
  http:           Remote repo http://server[;vrf]/remote_path
  https:          Remote repo https://server[;vrf]/remote_path
</code>
</pre>
</div>

**Info:** This install method doesn’t use local or remote repository feature, only a local TAR file copied on the router’s harddisk.
{: .notice--primary}

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#<mark>install package add source //harddisk:/8000-telnet-rpms.7.3.15.tar synchronous</mark>
Wed Aug 25 10:14:41.851 UTC
RP/0/RP0/CPU0:Aug 25 10:14:42.333 UTC: instorch[168]: %INFRA-INSTALL-6-ACTION_BEGIN : Packaging operation 1.1.1 started - add
Starting:
  install package add source /harddisk:/8000-telnet-rpms.7.3.15.tar
Packaging operation 1.1.1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing ..
Current activity: Veto check ..
Current activity: Package add or other package operation .^@.RP/0/RP0/CPU0:Aug 25 10:15:54.941 UTC: instorch[168]: %INFRA-INSTALL-6-ACTION_COMPLETE : Packaging operation 1.1.1 complete. More packaging operations can be performed, or any current changes can be applied by restart


Packaging operation 1.1.1: 'install package add source /harddisk:/8000-telnet-rpms.7.3.15.tar' completed without error
</code>
</pre>
</div>

Package is not active until it has been applied:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active summary
Wed Aug 25 10:16:33.691 UTC
Active Packages:    XR: 178    All: 1274
Label:              7.3.15
Software Hash:      9149973e08793c3f44e84a4a5b385c8a

Optional Packages                                                        Version
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

Second step is to apply the package. This is accomplished with *install apply* command:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#<mark>install apply synchronous</mark>
Wed Aug 25 10:17:35.066 UTC
Once the packaging dependencies have been determined, the install operation may have to reload the system.
If you want more control of the operation, then explicitly use 'install apply restart' or 'install apply reload' as reported by 'show install request'.
Continue? [yes/no]:[yes] <mark>yes</mark>
RP/0/RP0/CPU0:Aug 25 10:17:38.343 UTC: instorch[168]: %INFRA-INSTALL-6-ACTION_BEGIN : Apply by process restart 1.1 started
Starting:
  install apply restart
Atomic change 1.1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing
Current activity: Apply by restarting processes
RP/0/RP0/CPU0:Aug 25 10:17:42.502 UTC: schema_server[1183]: %INFRA-INSTALL-6-RESTART_DUE_TO_INSTALL : Process restarted due to install operation
 .....
RP/0/RP0/CPU0:Aug 25 10:18:35.618 UTC: instorch[168]: %INFRA-INSTALL-6-ACTION_COMPLETE : Apply by process restart 1.1 complete
RP/0/RP0/CPU0:Aug 25 10:18:35.621 UTC: instorch[168]: %INFRA-INSTALL-6-TRANSACTION_NOT_COMMITTED : The transaction is not committed. If the system reboots then the fallback software will be used. Use 'install commit' to commit the transaction and commit the active software.


Atomic change 1.1: 'install apply restart' completed without error 
</code>
</pre>
</div>

After this step, telnet package is active:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#sh install active summary
Wed Aug 25 10:18:44.681 UTC
Active Packages:    XR: 179    All: 1275
Label:              7.3.15
Software Hash:      d1072f4ad983408a49f6b99b37140998

Optional Packages                                                        Version
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
<mark>xr-telnet                                                         7.3.15v1.0.0-1</mark>
xr-track                                                          7.3.15v1.0.0-1 
</code>
</pre>
</div>

The last step is to commit current software configuration with *install commit* command. After this stage, active and committed packages match:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#<mark>install commit synchronous</mark>
Wed Aug 25 10:19:31.849 UTC
RP/0/RP0/CPU0:Aug 25 10:19:32.394 UTC: instorch[168]: %INFRA-INSTALL-6-ACTION_BEGIN : Commit 1 started
Starting:
  install commit
Transaction 1
Press Ctrl-C to return to the exec prompt. This will not cancel the install operation

Current activity: Initializing
Current activity: Commit transaction .RP/0/RP0/CPU0:Aug 25 10:19:48.190 UTC: instorch[168]: %INFRA-INSTALL-6-ACTION_COMPLETE : Commit 1 complete


Transaction 1: 'install commit' completed without error
RP/0/RP0/CPU0:8201-32FH-1#sh install committed summary
Wed Aug 25 10:20:10.960 UTC
Committed Packages: XR: 179    All: 1275
Label:              7.3.15
Software Hash:      d1072f4ad983408a49f6b99b37140998

Optional Packages                                                        Version
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
<mark>xr-telnet                                                         7.3.15v1.0.0-1</mark>
xr-track                                                          7.3.15v1.0.0-1 
</code>
</pre>
</div>

telnet package is now active and ready to use:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:8201-32FH-1#conf t
Wed Aug 25 10:24:05.131 UTC
RP/0/RP0/CPU0:8201-32FH-1(config)#tel
telemetry  <mark>telnet</mark>
RP/0/RP0/CPU0:8201-32FH-1(config)#telnet ?
  ipv4  IPv4 configuration
  ipv6  IPv6 configuration
  vrf   VRF name for telnet server
</code>
</pre>
</div>

## Conclusion
This article covered how to add a package on a Cisco 8000 running IOS XR7 architecture. While philosophy remains identical between classic IOS-XR, IOS-XR64bit and IOS XR7 (modular Network Operating Systems, ability to add and remove packages or software patches), few operational differences exist to add and activate a package.
