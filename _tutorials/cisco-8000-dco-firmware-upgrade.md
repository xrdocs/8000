---
published: true
date: '2024-07-02 11:31 +0200'
title: Cisco 8000 Optics Firmware Management
author: Frederic Cuiller
tags:
  - Cisco 8000
  - firmware
  - DCO
  - Bright ZR+
position: hidden
excerpt: >-
  This article aims at documenting QSFP-DD optical modules firmware upgrade on
  Cisco 8000 routers.
---

# Introduction

This Method of Procedure (MOP) document aims at documenting QSFP-DD optical modules software upgrade on Cisco 8000 routers.  
Like any Cisco 8000 systems’ components such route processor, line card, or even power supplies, optics also run embedded software. This software is used to configure, control and monitor the optic module. For Digital Coherent Optics (DCO), the Common Management Interface Specification (CMIS) interface is responsible for communication between the host (Cisco 8000 router) and the managed module (e.g Bright ZR+ DP04QSDD-HE0). 
While optics usually ships with a specific & stable firmware version from manufacturing, there are cases where it’s required to upgrade them. Such conditions include new functionality, software defect or interoperability fix.

# Initial State

In this example, we’ll upgrade a [Cisco 400G QSFP-DD Bright ZR+ module](https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/400g-qsfp-dd-high-power-optical-module-ds.html) from 70.120.14 to 70.130.21. The procedure described here is also applicable to different modules and 3rd party vendors.  

The following command helps to identify which firmware a module is currently running:

RP/0/RP0/CPU0:CISCO-82000#sh controllers Optics0/0/0/1
Thu Jun 27 22:36:14.025 JST
 Controller State: Up
 Transport Admin State: In Service
 Laser State: On
 Host Squelch Status: Enable
 LED State: Green
 FEC State: FEC ENABLED
 Power Mode: High
 Dom Data Status: Ready

 Optics Status

         Optics Type:  QDD 400G BRT ZRP
         DWDM carrier Info: C BAND, MSA ITU Channel=67, Frequency=192.80THz,
         Wavelength=1554.940nm
         Loopback Host : None
         Loopback Media : None

         Alarm Status:
         -------------
         Detected Alarms: None


         LOS/LOL/Fault Status:

         Alarm Statistics:

         -------------
         HIGH-RX-PWR = 0            LOW-RX-PWR = 0
         HIGH-TX-PWR = 0            LOW-TX-PWR = 0
         HIGH-LBC = 0               HIGH-DGD = 0
         OOR-CD = 0                 OSNR = 1
         WVL-OOL = 0                MEA  = 0
         IMPROPER-REM = 0
         TX-POWER-PROV-MISMATCH = 0
         Laser Bias Current = 234.3 mA
         Actual TX Power = 0.00 dBm
         Actual TX Power(mW) = 1.00 mW
         RX Power = -2.84 dBm
         RX Power(mW) = 0.51 mW
         RX Signal Power = -3.16 dBm
         Frequency Offset = -39 MHz
         Laser Temperature = 56.64 Celsius
         Laser Age = 0 %
         DAC Rate = 1x1.25

         Performance Monitoring: Enable

         THRESHOLD VALUES
         ----------------

         Parameter                 High Alarm  Low Alarm  High Warning  Low Warning
         ------------------------  ----------  ---------  ------------  -----------
         Rx Power Threshold(dBm)         13.0      -24.0          10.0        -21.0
         Rx Power Threshold(mW)          19.9        0.0          10.0          0.0
         Tx Power Threshold(dBm)          6.0      -18.0           4.0        -16.0
         Tx Power Threshold(mW)           3.9        0.0           2.5          0.0
         LBC Threshold(mA)               0.00       0.00          0.00         0.00
         Temp. Threshold(celsius)       83.00      -5.00         78.00        15.00
         Voltage Threshold(volt)         3.46       3.13          3.43         3.16

         LBC High Threshold = 98 %
         Configured Tx Power = 0.00 dBm
         Configured Tx Power(mW) = 1.00 mW
         Configured CD High Threshold = 160000 ps/nm
         Configured CD lower Threshold = -160000 ps/nm
         Configured OSNR lower Threshold = 9.00 dB
         Configured DGD Higher Threshold = 80.00 ps
         Baud Rate =  60.1385459900 GBd
         Modulation Type: 16QAM
         Chromatic Dispersion 0 ps/nm
         Configured CD-MIN -13000 ps/nm  CD-MAX 13000 ps/nm
         Second Order Polarization Mode Dispersion = 42.00 ps^2
         Optical Signal to Noise Ratio = 35.40 dB
         SNR = 19.40 dB
         Polarization Dependent Loss = 2.50 dB
         Polarization Change Rate = 0.00 rad/s
         Differential Group Delay = 4.00 ps

         Temperature = 63.00 Celsius
         Voltage = 3.29 V

 Transceiver Vendor Details

         Form Factor            : QSFP-DD
         Optics type            : QDD 400G BRT ZRP
         Name                   : CISCO-ACACIA
         OUI Number             : 7c.ffffffb2.5c
         Part Number            : DP04QSDD-HE0-190
         Rev Number             : A
         Serial Number          : ACA2740002C
         PID                    : DP04QSDD-HE0
         VID                    : V01
         Firmware Version       : Major.Minor.Build
                 Active         : 70.120.14
                 Inactive       : 70.120.14
         Date Code(yy/mm/dd)    : 23/09/14
         Fiber Connector Type: LC
         Otn Application Code: Undefined
         Sonet Application Code: Undefined

# Prerequisistes

The very first step is to get the new firmware. Contact your Cisco representative or Cisco TAC to get the new firmware.

Every firmware is provided with its own MD5 signature to verify file integrity:

rw-r--r--. 1 mliri     eng 1929216 May 15 09:11 gl2qsdd_gl2sfp8_70_130_21.ackit     <<< firmware file
-rw-r--r--. 1 mliri    eng      66 May 15 09:12 gl2qsdd_gl2sfp8_70_130_21.ackit.md5 <<< firmware signature

[06:49:42 fcuiller@people-new /auto/otbu-fpga/release_pending/flexcoh/QDD_400G_BRIGHT/firmware]$ cat gl2qsdd_gl2sfp8_70_130_21.ackit.md5
614a66be053114134fc92e3be301a02a  gl2qsdd_gl2sfp8_70_130_21.ackit


[deaccess@asj ~]$ md5sum gl2qsdd_gl2sfp8_70_130_21.ackit
614a66be053114134fc92e3be301a02a  gl2qsdd_gl2sfp8_70_130_21.ackit

Firmware must be copied to the Cisco 8000 router harddisk:

[deaccess@asj ~]$ scp gl2qsdd_gl2sfp8_70_130_21.ackit cisco@10.71.242.15:/harddisk:/
Password:
gl2qsdd_gl2sfp8_70_130_21.ackit                                                                                                                                                                                                       100% 1884KB   1.8MB/s   00:00
[deaccess@asj ~]$

File integrity is verified one more time from the router to make sure it’s not been corrupted in transit:

RP/0/RP0/CPU0:CISCO-8200#sh md5 file /harddisk:/gl2qsdd_gl2sfp8_70_130_21.ackit
Thu Jun 27 22:55:59.491 JST
614a66be053114134fc92e3be301a02a
RP/0/RP0/CPU0:CISCO-8200#


# Pre-upgrade tasks

It’s recommended to perform a sanity check before performing the firmware upgrade. While the list of KPI is customer dependent, the following information must be captured on both link ends:

sh controllers optics 0/0/0/1
sh controllers optics 0/0/0/1 pm
sh controllers  coherentDSP 0/0/0/1
sh controllers fourhundredGigE0/0/0/1 all
sh interfaces fourHundredGigE 0/0/0/1

In addition, Phil Bedard published a list of KPI to monitor in [this article]({{site.baseurl}}/design/blogs/xr-dco-monitoring).

Following CLI will provide a quick consolidated view:

RP/0/RP0/CPU0:CISCO-8200#sh controllers optics 0/0/0/1 summary
Thu Jun 27 23:02:54.260 JST

Port                Controller State        Admin State            LED State         Lane   Laser Bias   TX Power     RX Power     Wavelength     Optics Type       Description
                                                                                                          (dBm)        (dBm)        (nm)
------------------  ----------------------  ---------------------  ----------------  -----  ------------ -----------  ----------   -----------    --------------    -----------
Optics 0/0/0/1      Up                      In Service             Green             1      246.2mA      0.00         -2.60        1554.940       QSFP-DD-400G-BRIGHT-ZRP
RP/0/RP0/CPU0:CISCO-8200#

The idea of this sanity check is capture current optic state and compare it after the firmware upgrade.  

Once the sanity check is done, it’s recommended to drain/isolate the link. This can be achieved using regular routing techniques (ISIS overload bit, OSPF max metric, MPLS traffic engineering, BGP attributes, etc.).  

Optic module can be then administratively shutdown. This brings down associated ethernet ports and routing protocols:

RP/0/RP0/CPU0:CISCO-8200(config)#controller optics 0/0/0/1
RP/0/RP0/CPU0:CISCO-8200(config-Optics)#   shutdown
RP/0/RP0/CPU0:CISCO-8200(config-Optics)#commit
Thu Jun 27 23:14:49.640 JST
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.678 JST: ifmgr[337]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/0/0/1, changed state to Administratively Down
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.708 JST: config[66279]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'cisco'. Use 'show configuration commit changes 1000000261' to view the changes.
RP/0/RP0/CPU0:CISCO-8200(config-Optics)#RP/0/RP0/CPU0:2024 Jun 27 23:14:49.825 JST: npu_drvr[265]: %PLATFORM-VETH_PD-2-RX_FAULT : Interface FourHundredGigE0/0/0/1, Detected Local Fault
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.833 JST: bfd_agent[376]: %L2-BFD-6-SESSION_STATE_DOWN : BFD session to neighbor fe80::2 on interface FourHundredGigE0/0/0/1 has gone down. Reason: Control timer expired
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.836 JST: BM-DISTRIB[1188]: %L2-BM-5-MBR_BFD_SESSION_DOWN : The BFD session on link FourHundredGigE0/0/0/1 in Bundle-Ether1001 is down due to the receipt of a Down SCN. The member will be removed from the active members of the bundle.
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.836 JST: BM-DISTRIB[1188]: %L2-BM-6-ACTIVE : FourHundredGigE0/0/0/1 is no longer Active as part of Bundle-Ether1001 (BFD state of this link is Down)
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.836 JST: isis[1005]: %ROUTING-ISIS-5-ADJCHANGE : ISIS (100): Adjacency to CISCO-82000 (Bundle-Ether1001) (L2) Down, Interface state down
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.836 JST: ifmgr[337]: %PKT_INFRA-LINK-3-UPDOWN : Interface Bundle-Ether1001, changed state to Down
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.836 JST: ifmgr[337]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface Bundle-Ether1001, changed state to Down
RP/0/RP0/CPU0:2024 Jun 27 23:14:49.839 JST: bfd[1189]: %L2-BFD-6-SESSION_STATE_DOWN : BFD session to neighbor fe80::2 on interface Bundle-Ether1001 has gone down. Reason: Control timer expired
RP/0/RP0/CPU0:2024 Jun 27 23:14:50.026 JST: ifmgr[337]: %PKT_INFRA-LINK-3-UPDOWN : Interface FourHundredGigE0/0/0/1, changed state to Down
RP/0/RP0/CPU0:2024 Jun 27 23:14:50.026 JST: ifmgr[337]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface FourHundredGigE0/0/0/1, changed state to Down
RP/0/RP0/CPU0:2024 Jun 27 23:14:50.028 JST: BM-DISTRIB[1188]: %L2-BM-5-MBR_BFD_NOT_RUNNING : The BFD session on link FourHundredGigE0/0/0/1 in Bundle-Ether1001 is no longer required.


# Firmware upgrade

# Post-upgrade tasks

# Rollback

# Acknowledgement
I’d like to thanks Phil Bedard, Sreenivasa Rao Bandlamudi, Takumi Takiguchi & Marco Liri.
