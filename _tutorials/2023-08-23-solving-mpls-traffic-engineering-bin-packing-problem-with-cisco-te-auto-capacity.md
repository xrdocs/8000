---
published: false
date: '2023-08-23 10:13 +0200'
title: >-
  Solving MPLS Traffic Engineering Bin-Packing problem with Cisco TE++
  Auto-Capacity
tags:
  - iosxr
  - Cisco 80000
  - TE++
  - Auto-bandwidth Capacity
position: hidden
author: Frederic Cuiller
excerpt: >-
  This blog details Cisco’s TE++ Auto-bandwidth Capacity feature using a Cisco
  8000 environment.
---
# Introduction

Building point-to-point Traffic Engineering LSPs can lead to inefficient total network bandwidth utilization. Cisco’s TE++ Auto-bandwidth Capacity allows an ingress router to interact with the other routers in the network, locate additional available bandwidth and automatically create parallel LSPs to acquire all the necessary network resources possible until they are no longer needed.  This blog details one of the possible applications of this feature using a Cisco 8000 environment.

This blog details Cisco’s TE++ Auto-bandwidth Capacity feature using a Cisco 8000 environment.



