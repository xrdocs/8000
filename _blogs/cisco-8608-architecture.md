---
published: true
date: '2023-06-16 00:50 +0200'
title: Cisco 8608 Architecture
position: hidden
excerpt: >-
  This post will describe Cisco 8000 centralized system architecture: Cisco
  8608, based on Silicon One Q200 ASIC.
author: Chang Soo Lee
tags:
  - Cisco 8000
  - Centralized System
  - Silicon One
---
{% include toc icon="table" title="Cisco 8608 Architecture" %}

## Introduction
The Cisco 8608 series is a part of Cisco’s Centralized Architecture based chassis design. Centralized systems have redundant Route Processors (RP w/CPU), redundant Switch cards (SCs w/Cisco Silicon One Q200 NPU) and in-service replaceable modular port adapters (MPA). Unlike distributed architecture, the forwarding decisions on Centralized platforms are centrally performed at the RP/SC and runs Cisco IOS XR software.  

![Figure1.png]({{site.baseurl}}/images/Figure1.png)
           Figure 1. Front view of the Cisco 8608

## RP

## Switching Card

## MPA

## Redundancy

## Conclusion
