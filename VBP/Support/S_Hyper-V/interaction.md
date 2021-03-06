---
title: Interaction with Hyper-V
parent: Hyper-V
grand_parent: Supplemental
nav_order: 10
---

# Interaction with Hyper-V

## Change Block Tracking

From Hyper-V 2016 onwards Microsoft introduced a native change block tracking protocol named "Resilient Change Tracking" (RCT).

### Change block tracking on third party SMB implementation

Since Veeam own Change Block Tracking filter driver is not compatible with third party SMB implementations (as sometimes implemented on hyper converged infrastructures) it is advised to upgrade the cluster nodes to Hyper-V 2016 or higher to leverage Microsoft native RCT in such situations.

### Mixed clusters and Change Block Tracking

As migrating Hyper-V clusters from 2012 R2 to 2016 or higher can be done using the "rolling procedure" a Hyper-V cluster might temporary run different versions, impacting the CBT mechanism usage. Be aware that the transition time should be limited to days, during updating the cluster to 2016.

| Hosts OS           | VM Level     | Cluster Level | CBT                 |
|--------------------|--------------|---------------|---------------------|
| All 2012 R2        | lower than 8 | lower than 9  | Veeam filter driver |
| Mixed              | up to 8      | lower than 9  | No CBT              |
| All 2016 or higher | Lower than 8 | 9 or higher   | No CBT              |
| All 2016 or higher | 8 or higher  | 9 or higher   | Microsoft RCT       |

## Backup of Microsoft S2D hyper converged cluster

For Storage Spaces direct (S2D) Clusters only On-Host proxy mode is available because of the local storage used by S2D.

When configuring a hyper converged infrastructure based on Microsoft Storage Spaces Direct one limitation to know about is that a volume (CSV) hosting virtual machines is owned by a single node of the cluster at a given time. This implies that all IOs (including backup workload generated by all nodes) will be served by the single node owning the volume.

A good rule of thumb to avoid such potential bottleneck is to create a number of volumes (CSV) that equals the number of nodes in the cluster or is 2 times higher, spreading IOs servicing across all nodes.

---

## References

- [Veeam Helpcenter - Hyper-V Change Block Tracking](https://helpcenter.veeam.com/docs/backup/hyperv/changed_block_tracking.html)
