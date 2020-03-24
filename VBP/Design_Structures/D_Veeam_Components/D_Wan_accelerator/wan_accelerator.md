---
title: WAN Accelerator
parent: WAN Accelerator
grand_parent: Veeam Components Designs
nav_order: 10
---

# Veeam WAN Accelerator

By combining multiple technologies such as network compression, multi-threading, dynamic TCP window size, variable block size deduplication and global caching, WAN acceleration provides sufficient capability when the network bandwidth is low or limited when performing Backup Copy and Replication jobs. This technology is specifically designed to accelerate Veeam job. Any other WAN acceleration technology should be disabled for Veeam traffic.

Introduction and see link at bottom of page

## WAN Accelerator Placement


WAN accelerators work in pairs: one WAN accelerator should be deployed at the source side and another at the target site. Many to one scenario is also possible but needs accurate sizing and has higher system requirements.

## WAN Accelerator System requirements

The source WAN accelerator consumes a high amount of CPU and memory whilst re-applying the WAN optimized compression algorithm. Recommended system configuration is 4 CPU cores and 8 GB RAM. When using an existing Veeam Managed Server for Wan Acceleration which already has a role such as Veeam Backup & Replication Server, Proxy or Windows Repository ensure you have not overcommitted the CPUs on that host and there is enough resource for each role, otherwise the job will wait for a free CPU to continue.
The I/O requirements for the source WAN accelerator spikes every time a new VM disk starts processing, so the typical I/O pattern is made of many small blocks. Thus, it is recommended to deploy WAN accelerators on disk configurations with decent I/O performance, avoiding high latency spinning disks.

![*Source WAN accelerator streams*](./media/Streams.png)

## WAN Accelerator Sizing

## References
