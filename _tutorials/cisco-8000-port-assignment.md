---
published: true
date: '2022-03-01 22:07 +0100'
title: 'Port Assignments on Cisco 8100/8200 and Cisco 8800 Platforms '
position: hidden
tags:
  - iosxr
  - Cisco 8000
author: Chang Soo Lee
excerpt: >-
  This article will highlight how ports are allocated to IFG/Slice/NPU for each
  Cisco 8000 line card and systems.
---
## Introduction 

This memo will highlight how ports are allocated to IFG/Slice/NPU for each line card and system. Understanding port assignment will aid network design planning and, by enabling a deeper understanding of traffic flow, facilitate rapid, post-hoc system troubleshooting.  
Let’s review how this allocation is done, platform by platform and line card by line card. The following CLI is used to identify the port/IFG/Slice/NPU assignment: 