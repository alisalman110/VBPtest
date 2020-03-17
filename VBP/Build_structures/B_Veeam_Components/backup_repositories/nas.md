---
title: NAS Repositories
parent: Backup Repositories
grand_parent: B-Veeam_Components
nav_order: 20
---

# NAS Repositories

NAS Repositories include SMB and NFS repositories.

## SMB Share

When using a SMB share as target please check the following points

- SMB 3.x.x must be fully supported by the storage vendor or Windows Server (recommended Windows
  Server 2016+)
- To improve performance and reduce the latency impact, use one of the RDMA features Windows Server
  provides with SMB direct. RoCE or iWarp.
- a 10 Gibt/s network interface can be saturated with modern CPUs. If the repository is able to
  write faster than consider 40 Gbit/s connections between source and repository
- Try to avoid to many routing hops between source and Veeam Repository this will add latency and
  reduce your performance
- When an application writes data to an SMB share using WinAPI, it receives success for this I/O
  operation prematurely - right after the corresponding data gets put into the Microsoft SMB client's
  sending queue. If subsequently the connection to a share gets lost – then the queue will remain in
  the memory, and SMB client will wait for the share to become available to try and finish writing
  that cached data. However, if a connection to the share does not restore in a timely manner, the
  content of the queue will be lost forever.
