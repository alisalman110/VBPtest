---
title: Guest Interaction
parent: Anatomy
nav_order: 60
permalink: /anatomy/guestinteraction/
---

# Guest Interaction

Veeam tries to communicate to the guest OS first by using native methods over network (RPC for Windows and SSH for Linux guests) and falling back to hypervisor VIX or PowerShell-Direct when no network connection is possible. Check platform specific sections for details on hypervisor communication.

## Guest Interaction Proxy

Guest Interaction Proxy (GIP) will only work for Windows guests. Usage of GIP is advised in one of the following scenarios:

- Running Guest Interaction (GI) on many VMs at the same time - using multiple GIP will share the load
- Having network restrictions between VBR server and guest OS - using GIP limits the required firewall rules to the backup infrastructure
- Need to reduce the network load - GIP in a robo side will be used for copying the GI executables instead of VBR from HQ

## Reverse Guest Interaction Connection Order

The default order is

1. Connect to guest OS via network (RPC/SSH)
2. Connect to guest OS via hypervisor (VIX/PS-Direct)

This order can be reversed to save time, e.g. when network mode is prohibited and will always fail by configuration.

| Item  | Value                                                            |
| ----- | ---------------------------------------------------------------- |
| Path  | `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication` |
| Key   | `InverseVssProtocolOrder`                                        |
| Type  | REG_DWORD (32-bit)                                               |
| Value | `0` = try connection through RPC, failover to VIX (default)      |
| Value | `1` = try connection through VIX, failover to RPC                |

## File Exclusions

Veeam can exclude files from an image backup of a Windows guest and NTFS volume. This works similar to excluding deleted blocks or page files.

For each VM in a job that has exclusions enabled Veeam Backup and Replication performs the following operations: 

1. The virtual machine NTFS MFT is read into the memory cache on the backup proxy, data blocks that store excluded files are marked as deleted. 
2. When sending data blocks to the target repository data is read both from the VM snapshot and memory cache on the backup proxy. The target repository reconstructs the VM disks without the excluded VM data blocks. 
3. The virtual machine NTFS is modified using the data in the cache on the proxy and information about excluded data blocks is saved in the backup file or replica metadata. This information is necessary as CBT is not aware of which blocks were excluded and is used to determine which blocks should be processed during the next backup session.

---

## References

- [vSphere Guest Interaction](vmware/guestinteraction.md)
- [Hyper-V Guest Interaction](hyper-v/guestinteraction.md)
- [Veeam Helpcenter - Guest OS File Exclusion](https://helpcenter.veeam.com/docs/backup/vsphere/backup_job_vss_exclude.html)
