---
title: Backup Modes
parent: Hyper-V
grand_parent: Anatomy
nav_order: 20
---

# Hyper-V Backup Modes

Veeam Backup and Replication provides two different backup modes to process Hyper-V backups, both relying on the Microsoft VSS framework.

- **On-Host** backup mode, for which backup data processing is on the Hyper-V node hosting the VM, leveraging non transportable shadow copies by using software VSS provider.
- **Off-Host** backup mode, for which backup data processing is offloaded to another non-clustered participating Hyper-V node, leveraging transportable shadow copies using hardware VSS provider provided by the SAN storage vendor or SMB3 functionality.

Backup mode availability is heavily depending on the underlying virtualization infrastructure, leaving Off-Host backup mode available only to protect virtual machines hosted on SAN storage volumes or SMB3 compatible storage. It is important that the VSS framework provided by the storage vendor is tested and certified to work with Microsoft Hyper-V clusters. Intensive checks of vendor VSS provider during POC is highly recommended (Cluster environment).

Performance wise, since both backup modes are using the exact same Veeam transport services, the only differentiating factors will be the additional time requested to manage transportable snapshots (in favor of On-Host mode) and the balance between compute and backup resources consumption during backup windows (in favor of Off-Host mode).

When using Windows Server >= 2016 On-Host proxy mode is very fast and will reduce the included components and thus complexity. The Veeam Agent will then allocate the performance for deduplication and compression on the host systems. Consider that when planning the Veeam Job design. Please be aware that you will need up to 2GB of RAM on the Hyper-V Host per running task (one task = backup of one virtual disk). This memory must be free during Backup, otherwise the Hyper-V Host will start paging, what will end up in a slow system at all.

## Backup modes selection matrix

|              | PRO                                                                                                                                                                           | CON                                                                                                                                                                                                        |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **On-Host**  | <ul><li>Simplifies management</li><li>Does not require additional hardware, load is spread over all Hyper-V hosts</li><li>Can be used on any Hyper-V infrastructure</li></ul> | <ul><li>Requires additional resources from the hypervisors (CPU, Network IO and RAM) during the backup window, for IO processing and optimization</li><li>Scales with virtualization environment</li></ul> |
| **Off-Host** | <ul><li>No impact on the compute resources on the hosting Hyper-V Server</li></ul>                                                                                            | <ul><li>Adds additional delay for snapshots transportation</li><li>Requires compatible SAN or SMB3 storage</li><li>Might depend on third party VSS provider</li></ul>                                      |

## Limiting the impact of On-Host backup mode on the production infrastructure

While consuming production resources for backup purpose the On-Host backup mode disadvantages can be mitigated by the following guidelines.

- **Spreading load across hypervisors**. It should be kept in mind that the backup load, instead of being carried by a limited number of dedicated proxies, will be spread over all Hyper-V hosts. When needing 16 proxy task slots to meet the backup window, in a cluster of 8 hosts each hosting the same amount of proteced VMs will result in a task limit configuration of 2 per host thus limiting the impact per host.

- **Leveraging storage latency control**. This feature allows to protect the volumes (globally for Enterprise edition, and individually for Enterprise Plus edition) from high latency, by monitoring and adjusting backup load accordingly. Please refer to [user guide](https://helpcenter.veeam.com/docs/backup/hyperv/options_parallel_processing.html) proper section for further information.

---

## References
- [Veeam Helpcenter - Backup I/O Settings](https://helpcenter.veeam.com/docs/backup/hyperv/options_parallel_processing.html)
- [Veeam Helpcenter - Off-Host Proxy](https://helpcenter.veeam.com/docs/backup/hyperv/offhost_backup_proxy.html)
- [Veeam Helpcenter - Hyper-V Deployment Scenarios](https://helpcenter.veeam.com/docs/backup/hyperv/deployment_scenarios.html)