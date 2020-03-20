---
title: WAN Accelerator
parent: Veeam Components Designs
grand_parent: Design
nav_order: 10
---

# Veeam WAN Accelerator

Introduction and see link at bottom of page

By combining multiple technologies such as network compression, multi-threading, dynamic TCP window size, variable block size deduplication and global caching, WAN acceleration provides sufficient capability when the network bandwidth is low or limited when performing Backup Copy and Replication jobs. This technology is specifically designed to accelerate Veeam job. Any other WAN acceleration technology should be disabled for Veeam traffic.


## WAN Accelerator Placement
WAN accelerators work in pairs: one WAN accelerator should be deployed at the source side and another at the target site. Many to one scenario is also possible but needs accurate sizing and has higher system requirements.

## WAN Accelerator System requirements

![*Source WAN accelerator IOPS*](./Media/Source_WAN_IOPS.png)

## WAN Accelerator Sizing

## References