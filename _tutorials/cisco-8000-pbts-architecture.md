---
published: true
date: '2022-12-13 18:14 +0100'
title: Cisco 8000 PBTS Architecture -- edited
author: Yosef Manasseh
excerpt: This post aims to describe PBTS implementation on Cisco 8000 routers.
tags:
  - iosxr
  - Cisco 8000
  - PBTS
position: hidden
---
## A New Post -- edited

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

Feature Description

PBTS (Policy Based Tunnel Selection) is a feature where operator can classify incoming network traffic and forward that traffic to destination via specific MPLS traffic engineering tunnels (TE tunnel).
PBTS implementation on Cisco 8000 described in this article classifies the incoming traffic on ingress interface based on EXP field (for MPLS traffic) or prec/DSCP field (for IP traffic). Thus it is possible, for instance, to forward traffic with specific EXP/Prec/DSCP value via a specific TE tunnel.

![01_overview.png]({{site.baseurl}}/images/01_overview.png)
