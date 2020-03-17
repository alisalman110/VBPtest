---
title: Guest Interaction
parent: Operational
grand_parent: Anatomy
nav_order: 10
---

# Guest Interaction

Veeam tries to communicate to the guest OS first by using native methods over network (RPC for
Windows and SSH for Linux guests) and falling back to hypervisor VIX or PowerShell-Direct when no
network connection is possible. Check platform specific sections for details on hypervisor
communication.

RPC connection means injecting the file via the "ADMIN\$" share on the target VM. See the
[Veeam KB1230](https://www.veeam.com/kb1230) for more information. Consider that this is a global
setting that will be applied on the Veeam backup server level and affects all jobs with
application-aware image processing.

## Guest Interaction Proxy

Depending on the guest VM operating system and/or Veeam Backup and Replication Edition different
servers may be selected to perform guest processing step and initiate connection to a VM as per the
table below.

| Edition         | Windows                 | Linux         |
| --------------- | ----------------------- | ------------- |
| Standard        | Backup server           | Backup server |
| Enterprise      | Guest interaction proxy | Backup server |
| Enterprise Plus | Guest interaction proxy | Backup server |

Usage of GIP is advised in one of the following scenarios:

- Running Guest Interaction (GI) on many VMs at the same time - using multiple GIP will share the load
- Having network restrictions between VBR server and guest OS - using GIP limits the required firewall rules to the backup infrastructure
- Need to reduce the network load - GIP in a robo side will be used for copying the GI executables instead of VBR from HQ

Any Windows server managed by Veeam Backup and Replication can be selected to act as guest
interaction proxy but the preference would be given to the server that has IP address in the same
subnet as subject VM. This functionality allows for having only small limited range of ports to
allow through the firewalls in restricted environments and for that reason it is recommended to have
guest interaction proxies in all VM subnets that are not supposed to be directly accessible from the
network where Veeam backup server resides.

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
