---
published: true
date: '2023-07-11 11:46 +0200'
title: 'Cisco 8800 Fan Tray, Fabric Card & Line Card Replacement Procedure'
excerpt: >-
  This blog post details the steps required to replace a FT, FC and LC on a
  Cisco 8800 router. 
position: hidden
tags:
  - cisco 8000
  - Hardware
  - Method of Procedure
author: Manish Chandra Pandey
---
## Introduction

On a Cisco 8800 router, Fan Tray (FT), Fabric Cards (FC) and Line Cards (LC) are Field Replaceable Units (FRU).  
8800 distributed systems use four Fan Trays but can operate with three fan trays while operator replaces one, or temporarily remove one to replace one of the Fabric Cards sitting behind the Fan Tray.  When a Fan Tray is removed, the remaining Fan Trays speed up their fans to maintain the designed airflow.  
Fabric Cards sit behind the Fan Trays on the chassis and the operator must remove Fan Tray unit to replace a Fabric card. 
This blog post details the steps required to replace a FT, FC and LC on a Cisco 8800 router. A video with detailed steps for FT and FC replacement is also linked in this post below and takes  8818 chassis as an example. At the time of publishing this video, the fabric card rail extension is only available on 8818 routers. All other steps are applicable to other chassis (8804, 8808, 8812).

## Before Getting Started

Verify the following –
- Line card inserted
'show platform'
- IOS XR version
'show install active summary'
'show install committed summary'
- FPD status of the cards
'show hw-module fpd' (should be “CURRENT”)


## Fan Tray/Fabric Card Replacement

Fabric cards mount vertically into the chassis connecting orthogonally to the horizontally placed line cards in the system. Fabric cards utilize ejectors on them to gracefully handle card insertion and removal from the chassis. Ejector handles should be operated properly as directed in this article/video for smooth and uninterrupted functioning of the card in the system.

## Line Card Replacement
### Line Card Removal
### Line Card Insertion

## General Verification

## Conclusion

This concludes the steps required for replacement of FT, FC and LC on 8800 system. Following meticulously to the outlined steps provided in this document will result in smooth operations.



