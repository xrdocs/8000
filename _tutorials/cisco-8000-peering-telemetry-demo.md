---
published: true
date: '2024-03-07 14:46 +0100'
title: Cisco Live Melbourne 2023 – Cisco 8000 Peering Telemetry Demo
author: Frederic Cuiller
tags:
  - Cisco 8000
  - Peering
  - Telemetry
position: top
excerpt: >-
  This video aims to list the requirements of an Internet peering router. It
  demonstrates how Cisco 8000 series routers, powered by Silicon One and IOS XR
  can meet them. The video shows live examples of telemetry collection: RPKI,
  FIB, Netflow, BGP, QoS, uRPF/RTBH/SRTBH and BGP Flowspec.
---
# Introduction

This video aims to list the requirements of an Internet peering router. It demonstrates how Cisco 8000 series routers, powered by Silicon One and IOS XR can meet them.  
The video shows live examples of telemetry collection: RPKI, FIB, Netflow, BGP, QoS, uRPF/RTBH/SRTBH and BGP Flowspec.

# Demo Details

This silent/audio-less video was recorded for Cisco Live Melbourne 2023 and was the support of a World of Solutions demo.  

The test is done on a Cisco 8201-32FH fixed system running IOS XR 7.10.1. It aims to represent a realistic Internet peering router, including:
- multiple BGP sessions to simulate PNI and Transit, all dual-stack
- typical Internet peeering router feature-set and scale
- industry BCOP (Best Current Operational Practices) configured and deployed
- a DDoS mitigation platform (emulated with a BGP daemon to craft BGP and BGP-FS updates)

# Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/iEgYLguc4us?si=wuDjeNhtsOkpX9me" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# Telemetry Configuration

The configuration used for this demo can be found below. It must be tuned and adapted before being deployed: the sampling interval is sometimes very agressive and is only relevant for a lab application.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
telemetry model-driven
 destination-group TME-JUMP
  address-family ipv4 1.63.51.21 port 57500
   encoding self-describing-gpb
   protocol grpc no-tls
  !
 !
 sensor-group OFA
  sensor-path Cisco-IOS-XR-platforms-ofa-oper:ofa
 !
 sensor-group BGP-COUNTERS
  sensor-path openconfig-bgp:bgp/global/state
  sensor-path Cisco-IOS-XR-ipv4-bgp-oper:bgp/bpm-instances-table/bpm-instances
  sensor-path Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/default-vrf/neighbors/neighbor
  sensor-path Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/default-vrf/afs/af/af-process-info
  sensor-path Cisco-IOS-XR-ipv4-bgp-oc-oper:oc-bgp/bgp-rib/afi-safi-table/ipv4-unicast/open-config-neighbors/open-config-neighbor/adj-rib-in-post/num-routes/num-routes
  sensor-path Cisco-IOS-XR-ipv4-bgp-oc-oper:oc-bgp/bgp-rib/afi-safi-table/ipv6-unicast/open-config-neighbors/open-config-neighbor/adj-rib-in-post/num-routes/num-routes
 !
 sensor-group FIB-COUNTERS
  sensor-path Cisco-IOS-XR-fib-common-oper:fib/nodes/node/protocols/protocol/vrfs/vrf/summary
 !
 sensor-group QOS-COUNTERS
  sensor-path Cisco-IOS-XR-qos-ma-oper:qos/interface-table/interface[interface-name='FourHundredGigE0/0/0/26']/input/service-policy-names/service-policy-instance[service-policy-name='RATE-LIMIT-DDOS']/statistics/class-stats
 !
 sensor-group RIB-COUNTERS
  sensor-path Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count
  sensor-path Cisco-IOS-XR-ip-rib-ipv6-oper:ipv6-rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count/active-routes-count
 !
 sensor-group RPKI-COUNTERS
  sensor-path Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/rpki-summary
 !
 sensor-group URPF-COUNTERS
  sensor-path Cisco-IOS-XR-fib-common-oper:fib-statistics/nodes/node/drops
 !
 sensor-group BGPFS-COUNTERS
  sensor-path Cisco-IOS-XR-flowspec-oper:flow-spec/summary
  sensor-path Cisco-IOS-XR-flowspec-oper:flow-spec/vrfs/vrf/afs/af/table-summary
  sensor-path Cisco-IOS-XR-flowspec-oper:flow-spec/vrfs/vrf/afs/af/flows/flow/flow-statistics
 !
 sensor-group MEMORY-COUNTERS
  sensor-path Cisco-IOS-XR-nto-misc-oper:memory-summary/nodes/node/detail
  sensor-path Cisco-IOS-XR-procmem-oper:processes-memory/nodes/node/process-ids/process-id
 !
 sensor-group NETFLOW-COUNTERS
  sensor-path Cisco-IOS-XR-ofa-netflow-oper:net-flow/statistics/statistic/server/flow-exporters/flow-exporter/exporter
 !
 sensor-group BGP-PROCESS-COUNTERS
  sensor-path Cisco-IOS-XR-procmem-oper:processes-memory/nodes/node[node-name='0/RP0/CPU0']/process-ids/process-id[process-id='10996']/process-id
 !
 sensor-group SYSTEM-CPU-MEMORY-COUNTERS
  sensor-path Cisco-IOS-XR-wdsysmon-fd-oper:system-monitoring
  sensor-path Cisco-IOS-XR-wdsysmon-fd-oper:system-monitoring/cpu-utilization
  sensor-path Cisco-IOS-XR-nto-misc-oper:memory-summary/nodes/node/summary
 !
 subscription CONVERGENCE
  sensor-group-id OFA sample-interval 1000
  sensor-group-id BGP-COUNTERS sample-interval 1000
  sensor-group-id FIB-COUNTERS sample-interval 1000
  sensor-group-id RIB-COUNTERS sample-interval 1000
  destination-id TME-JUMP
 !
 subscription SUBSCRIPTION
  sensor-group-id OFA sample-interval 1000
  sensor-group-id BGP-COUNTERS sample-interval 1000
  sensor-group-id FIB-COUNTERS sample-interval 1000
  sensor-group-id QOS-COUNTERS sample-interval 30000
  sensor-group-id RIB-COUNTERS sample-interval 1000
  sensor-group-id RPKI-COUNTERS sample-interval 30000
  sensor-group-id URPF-COUNTERS sample-interval 30000
  sensor-group-id BGPFS-COUNTERS sample-interval 30000
  sensor-group-id MEMORY-COUNTERS sample-interval 30000
  sensor-group-id NETFLOW-COUNTERS sample-interval 30000
  sensor-group-id SYSTEM-CPU-MEMORY-COUNTERS sample-interval 30000
  destination-id TME-JUMP
</code>
</pre>
</div>

# Telegraf Configuration

Here is the relevant part of telegraf.conf file, containing the embedded_tags:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
[[inputs.cisco_telemetry_mdt]]
   transport = "grpc"
   service_address = ":57500"

#  ## Define (for certain nested telemetry measurements with embedded tags) which fields are tags

   embedded_tags = ["Cisco-IOS-XR-lpts-pre-ifib-oper:lpts-pifib/nodes/node/pifib-hw-flow-policer-stats/police-info/flow-name","Cisco-IOS-XR-lpts-pre-ifib-oper:lpts-pifib/nodes/node/pifib-hw-flow-policer-stats/police-info/npu-id", "Cisco-IOS-XR-fib-common-oper:fib/nodes/node/protocols/protocol/fib-summaries/fib-summary/prefix-masklen-distribution/unicast-prefix/mask-length", "Cisco-IOS-XR-platforms-ofa-oper:ofa/stats/nodes/node/Cisco-IOS-XR-8000-platforms-npu-resources-oper:hw-resources-datas/hw-resources-data/npu-hwr/npu-id", "Cisco-IOS-XR-platforms-ofa-oper:ofa/stats/nodes/node/Cisco-IOS-XR-8000-platforms-npu-resources-oper:hw-resources-datas/hw-resources-data/npu-hwr/bank/bank-name", "Cisco-IOS-XR-platforms-ofa-oper:ofa/stats/nodes/node/Cisco-IOS-XR-8000-platforms-npu-resources-oper:hw-resources-datas/hw-resources-data/npu-hwr/bank/lt-hwr/name", "Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/default-vrf/neighbors/neighbor/af-data/af-name", "Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/name", "Cisco-IOS-XR-procmem-oper:processes-memory/nodes/node/process-ids/name", "Cisco-IOS-XR-qos-ma-oper:qos/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/class-name"]
</code>
</pre>
</div>

# Grafana Dashboard

The Grafana dashboard used for the demo has been exported in JSON and uploaded into the [xrdocs.io GitHub repo](https://raw.githubusercontent.com/xrdocs/8000/gh-pages/8000%20Peering%20Telemetry%20Dashboard.json). It can be imported in your own environment. Following parameters must be adapted:
- datasource: you must put your own TSDB, configured in Grafana
- source: you must put your exact device hostname

# Conclusion

Cisco 8000 is a great fit for peering role. It offers multiple form factors to cover all the capacity needs. With more than 20 year of field exposure in carrier-class environment, IOS XR BGP stack is fast, scalable, reliable and secure.  Cisco Silicon One feature-set and associated scale is complete for this network function. Last, as demonstrated in the video, IOS XR YANG modules can provide full visibility on multiple critical components used for this role.
