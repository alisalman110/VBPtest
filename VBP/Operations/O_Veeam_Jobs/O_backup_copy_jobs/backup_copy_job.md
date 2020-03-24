---
title: Backup Copy Job General
parent: Veeam Backup Copy Job
grand_parent: Operate
nav_order: 10
---

# Backup Copy Job

## Job Layout and Object Selection

### Source Object Container

- **Select from infrastructure**: this selects specific VMs or containers from the virtual infrastructure. The scheduler will look for the most recent restore point containing the VMs within the synchronization interval. The scheduler will look for restore points in all backups, regardless which job generated the restore point. If the restore point is locked (e.g. the backup job creating it is running), the backup copy job waits for the restore point to be unlocked and then start copying the state of the VM restore point according to its defined schedule.

- **Select from job**: this method of selection is very useful if you have multiple backup jobs protecting the same VMs. In this case, you can bind the backup copy job to a specific job you want to copy. The job container will protect all the VMs in the selected source job(s).

- **Select from backup**: this method is equivalent to the Select from infrastructure method, but allows for selecting specific VMs inside specific backups. This is helpful, when only certain critical VMs should be copied offsite.

### Backup Copy and Tags

As you can select any VM to be copied from multiple backups, you can plan for policy-based configurations. For instance, you may not want to apply GFS retention over some VMs like web servers, DHCP, etc. In this situation, you can use VMware tags to simplify the management of backup copy process. Tags can be easily defined according to the desired backup copy configuration, using VMware vSphere or Veeam ONE Business View to apply tags.

## Initial synchronization

When creating the initial copy to the secondary repository, it is recommended to use backup seeding (see Creating Seed for Backup Copy Job) whenever possible. Especially when transferring large amounts of data over less performant WAN links, the seeding approach can help mitigating initial synchronization issues.

While Backup Copy Jobs were designed for WAN resiliency, the initial copy is more error prone, as it is typically transferring data outside the datacenter over less reliable links (high latency, or packet loss). Another issue that can be solved by seeding is when the full backup is larger than the amount of data that can be transferred in an interval. Even if the interval can be extended to accomodate the initial transfer, this may lead to upload times of even multiple days. Seeding can speed up the initial sync by removing the need for the sync.

## Additional options

### Restore point lookup

By default, after a restart of the job interval (the Copy every setting), a backup copy job analyzes the VM list it has to protect, and searches backwards in time for newer restore point states. If the state of the restore point in the target repository is older than the state in the source repository, the new state is transferred.

To change this behavior, it is possible to use the `BackupCopyLookForward` registry key as described below. Reevaluating the example above, using this registry key, the backup copy job will still start searching at 10:00 PM, but will now wait for a new restore point state created after this point in time.

|       |                                                                                                    |
| ----- | -------------------------------------------------------------------------------------------------- |
|  Path | `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`                                   |
|   Key | `BackupCopyLookForward`                                                                            |
|  Type | REG_DWORD (32-bit)                                                                                 |
| Value | **`1`** = _enable the option_                                                                      |

<hr>

### References

[Backup Copy Job](https://helpcenter.veeam.com/docs/backup/vsphere/backup_copy.html?ver=100)

[Creating Seed for Backup Copy Job](https://helpcenter.veeam.com/docs/backup/vsphere/backup_copy_mapping_auxiliary.html?ver=100)

[Mapping backup files to a backup copy job](https://helpcenter.veeam.com/docs/backup/vsphere/backup_copy_mapping_file.html?ver=100)