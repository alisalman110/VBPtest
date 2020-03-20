---
title: Hyper-V backup proxies
parent: Veeam Backup Proxies
grand_parent: Veeam Components
nav_order: 10
---

# Hyper-V backup modes #

Veeam Backup and Replication provides two different backup modes to process Hyper-V backups, both relying on the Microsoft VSS framework.
- **On-Host** backup mode, for which backup data processing is on the Hyper-V node hosting the VM, leveraging non transportable shadow copies by using software VSS provider.
- **Off-Host** backup mode, for which backup data processing is offloaded to another non clustered participating Hyper-V node, leveraging transportable shadow copies using Hardware VSS provider provided by the SAN storage vendor.

Backup mode availability is heavily depending on the underlying virtualization infrastructure, leaving Off-Host backup mode available only to protect virtual machines hosted on SAN storage volumes or SMB3 shares.

Performance wise, since both backup modes are using the exact same Veeam transport services, the only differentiating factors will be the additional time requested to manage transportable snapshots (in favor of On-Host mode) and the balance between compute and backup resources consumption during backup windows (in favor of Off-Host mode).

**Backup modes selection matrix**

|   |PRO|CON|
|---|---|---|
|**On-Host**|Simplifies management.  Does not depend on third party VSS provider.  Does not require additional hardware.  Can be used on any Hyper-V infrastructures.|Requires additional resources from the hypervisors during the backup window, for IO processing and optimization.  Does not depend on third party VSS provider.  Does not require additional hardware.|
|**Off-Host**|No impact on the compute resources on the hosting hyper-v.  Requires third party VSS hardware provider.|Adds additional delay for snapshots transportation.  Available only for virtualization infrastructures based on SAN storage or SMB3 shares.|

## Limiting the impact of On-Host backup mode on the production infrastructure ##

While consuming production resources for backup purpose the On-Host backup mode disadvantages can be mitigated by the following guidelines.
- **Spreading load across hypervisors**. It should be kept in mind that the backup load, instead of being carried by a limited number of dedicated proxies, will be spread through all the hypervisors. Default Veeam setting is to limit backup to 4 parallel tasks per hypervisor, which will use a maximum of four cores and 8 GB of RAM. This can be modified in the “Managed server” section of the Veeam Console, through the “Task limit” setting. For instance, if the sizing guidelines results in a total amount of 24 cores and 48 GB of RAM needed for Veeam transport services, and the infrastructure comprises 12 Hyper-V servers, each server task limit can be set to 2.

# Related links #
[On-host backup for Hyper-V](https://helpcenter.veeam.com/docs/backup/hyperv/onhost_backup.html?ver=100)

[Off-host backup for Hyper-V](https://helpcenter.veeam.com/docs/backup/hyperv/offhost_backup.html?ver=100)

[Off-host proxy on Hyper-V](https://helpcenter.veeam.com/docs/backup/hyperv/offhost_backup_proxy.html?ver=100)
