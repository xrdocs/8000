---
published: true
date: '2022-07-07 07:47 -0700'
title: '8000 Emulator Notebooks: Documentation at Runtime'
author: Cisco MIG Notebooks Team
excerpt: Cisco 8000 Emulator integrated with Jupyter Notebooks
tags:
  - iosxr
  - cisco
  - '8000'
  - Jupyter
  - Python
position: hidden
---
Are you looking for a programmatically accurate document to guide you through your specific configuration needs?
 
Also, simulate a use case to test-drive, even before actual deployments? 
 
That enables you to perform your task effortlessly and provides a seamless experience between the written word and the actual deployment? 
 
You've come to the right place!

Cisco 8000 Emulator Notebooks provide a real-time configuration experience— where you can simulate and configure the device as you read through the text and view the output instantly.

 
Read on to know more!

## What are Cisco 8000 Emulator Notebooks?
The Cisco 8000 Emulator Notebooks are live documents that combine narrative text, images, videos, runnable code, and real-time outputs. 
 
These notebooks are created using Jupyter, an open source software.

## How do these Notebooks work?
The Notebook communicates with the Cisco 8000 emulator running in the background and brings up multi-router topologies within minutes at the click of the play button. You can then execute commands and configurations on the emulated routers directly from the Notebook. 

![NBUI.png]({{site.baseurl}}/images/NBUI.png)

## Why Notebooks?
The PyVxr API, when invoked from within Notebooks, allows you to build and manage virtual multi-router topologies within minutes.
 
The PyVxr API sends your configurations to the virtual routers and retrieves real-time status in the same document. You can read the content, run, customize, and experiment with configurations based on your business needs.
![NBUex.png]({{site.baseurl}}/images/NBUex.png)

## What are the Benefits?
* Automates the creation and bringing-up of the topology on the Cisco 8000 Emulator, and saves time.
* Provides flexibility to customize topology by adding and modifying CLIs.
* Enables test proof-of-concept, use cases, or features to existing problems.
* Doubles up as a tutorial to educate customers or partners.
* Allows use case simulation and testing before actual deployments

## Access Cisco 8000 Emulator Notebooks
*Click here to view* [Cisco 8000 Emulator Notebooks](https://github.com/ios-xr/network-notebooks)

## Your Notebook Journey Begins Here
### Getting Started
_1. How to use Notebooks_

* [Notebook User Interface](https://github.com/ios-xr/network-notebooks/blob/main/Getting-Started/How-to-use-Notebooks/Using_Notebooks.ipynb)

* [Interaction with Cisco 8000 Emulator](https://github.com/ios-xr/network-notebooks/blob/main/Getting-Started/How-to-use-Notebooks/Notebook-magic.ipynb) 

* [Using Pyvxr Test Framework within Notebooks](https://github.com/ios-xr/network-notebooks/blob/main/Getting-Started/How-to-use-Notebooks/Test-framework.ipynb)


_2. Setting up a basic network_

* [Configure a Small Core Network](https://github.com/ios-xr/network-notebooks/blob/main/Getting-Started/Setting-Up-Basic-Network/Small-Core/SmallCore.ipynb)

* [Configure a Label Switched Network](https://github.com/ios-xr/network-notebooks/blob/main/Getting-Started/Setting-Up-Basic-Network/Ospf-Mpls/ospf-mpls.ipynb)

### Put Technology to Work
Explore end-to-end use cases

_Traffic Management_

* [QoS, to make Networks Dependable](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/QoS/QoS-policies.ipynb)

* [Prioritize Delay-Sensitive Traffic Using QOS](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/QoS/QoS-high-priority-flows.ipynb)

* [Filter Network Traffic Using Access Control List ](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/ACL/Access-Control-List.ipynb)

* [Filter Network Traffic Using ACL Yang ](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/ACL/ACLyang.ipynb)

_Secure Network_

* [Type 6 Password Authentication For BGP](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/BGP-Type6-Password-Encryption/Type6-BGP-4-Routers.ipynb)

_Network Monitoring_

* [Monitor Traffic Using Encapsulated Remote Switched Port Analyzer (ERSPAN)](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/ERSPAN/erspan.ipynb)

* [Stream CPU Utilization Data Using Model-driven Telemetry](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/Telemetry/TelemetryDialOutCLI.ipynb)

* [Deploy YANG Data Model to Stream CPU Utilization Data Using Model-driven Telemetry](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/Telemetry/TelemetryDialOutYang.ipynb)

* [Setup a Pipeline and Stream CPU Utilization Data Using Model-driven Telemetry](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/Telemetry/TelemetryDialOutGPB-Yang.ipynb)

_APIs_

* [Super-charge Your Router Performance With Service-layer APIs](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/ServiceLayer/ServiceLayerAPI.ipynb) 

_Utilities_

* [Python Library to Generate Real-time Traffic On Simulated Networks](https://github.com/ios-xr/network-notebooks/blob/main/Put-Technology-to-Work/trafficUtils/README.md) 

## Try out these use cases
You can configure and try out these use cases on emulated Cisco 8000 routers using Cisco 8000 Emulator Notebooks. 
For real-time configuration experience, Contact your Cisco Account Manager to download the Cisco 8000 Emulator software package and then Install the [Cisco 8000 Hardware Emulator](https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000-emulator/cisco8000-hardware-emulator-installation-guide.html).

Reach us at [mig-notebooks@cisco.com]() for feedback or queries about these use cases
