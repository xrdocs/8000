---
published: false
date: '2023-09-02 21:09 +0530'
title: >-
  New-Cisco 8000 Hardware resources deep dive Part-1: Encapsulation database for
  label transport
---
{% include toc icon="table" title="Cisco 8000 Hardware resources deep dive Part-1: Encapsulation database for label transport" %}

|Ram Mohan A.M., Technical Leader, Technical Marketing (raam@cisco.com)|  
|Deepak Balasubramanian, Technical Leader, Technical Marketing (deebalas@cisco.com)|  

## Introduction  

With Cisco 8000 routers gaining popularity across various service providers and cloud customers, it’s very important to understand how the system behaves when running labelled transport and services. The reason being the labelled applications are hardware resource intensive, especially on the systems built with single stage forwarding architecture.

In this article we will focus on the basic label forwarding transport with & without ECMP along with bundles and look into details on the hardware resource usage. Main focus for this article will be the Encapsulation Database.

Also, in this article we will focus on the Silicon One **Q100 and Q200 based 8200/8600/8800** systems only.
Resource utilizations for the next generation of Silicon One Asics (P100, K100) are different and we will have another XR docs article focusing on that.


