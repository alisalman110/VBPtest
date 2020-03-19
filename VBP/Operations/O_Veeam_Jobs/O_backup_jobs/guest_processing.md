---
title: Guest Processing
parent: Veeam Backup Jobs
grand_parent: Veeam Jobs
nav_order: 50
---

# Application-Aware Image Processing / Guest Processing

When configuring backup and replication jobs, you can specify how to create the
transactionally-consistent backup images of VMs. Two methods are available for bringing VM file
system and applications into consistent state: VMware Tools/Hyper-V Integration Services quiescence
or Veeam's proprietary application-aware image processing (using Microsoft VSS or Linux scripts).
Key features of both methods are illustrated by the following table:

| Feature                                                                           | VMware Tools Quiescence  | Hyper-V Quiescence    | Application-Aware Image Processing  |
| --------------------------------------------------------------------------------- | ------------------------ | --------------------- | ----------------------------------- |
| Support for consistent backup on Windows guest                                    | Yes                      | Yes                   | Yes                                 |
| Sync driver for Linux guest                                                       | Yes                      | No                    | No                                  |
| Support for application-aware backup                                              | Limited                  | Yes                   | Yes                                 |
| Pre-VSS preparation for specific applications (e.g. Oracle)                       | No                       | No                    | Yes                                 |
| Support for application log truncation (Microsoft SQL Server and Exchange Server) | No                       | No                    | Yes                                 |
| Support for scripts                                                               | Yes (placed on VM guest) | Yes (placed on guest) | Yes (can be centrally distributed)  |
| Error reporting                                                                   | Within VM guest OS       | Within VM guest OS    | Centralized, on Veeam backup server |

## Selecting Guest Processing Options

When on the **Guest Processing** step of the job wizard, you are presented with the variety of
options (as described in detail in the User Guide for [vSphere](https://helpcenter.veeam.com/docs/backup/vsphere/backup_job_vss_vm.html)
and [Hyper-V](https://helpcenter.veeam.com/docs/backup/hyperv/backup_job_vss_hv.html)).

Note that you can use pre- and post-job scripting to automate job global settings from the Veeam
Backup & Replication server itself. It is recommended to use the VM guest processing options for
interaction with VMs.

To select the necessary options, refer to the table below.

| VM guest OS type                                                    | Linux (with applications and known user for Guest OS processing) | Windows and VMware VSS-supported applications (without known user for Guest OS processing) | Windows with VSS-aware applications | Windows (no VSS-aware applications) | Linux with applications | Linux server (no applications) |
| ------------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------- | ----------------------------------- | ----------------------- | ------------------------------ |
| Guest OS processing is applicable                                   | Y                                                                | Y                                                                                          | Y                                   | Y                                   | Y                       | N                              |
| Use hypervisor quiescence                                           | N                                                                | Y                                                                                          | N                                   | N                                   | N                       | N                              |
| Hypervisor quiescence with native script processing                 | Y                                                                | N                                                                                          | N                                   | N                                   | N                       | N                              |
| Enable Veeam Application-Aware Image Processing                     | N                                                                | N                                                                                          | Y                                   | N                                   | N                       | N                              |
| Enable Veeam Application-Aware Image Processing and InGuest Scripts | N                                                                | N                                                                                          | N                                   | Y                                   | N                       | N                              |
| Disable Veeam Application-Aware Image Processing                    | N                                                                | N                                                                                          | N                                   | N                                   | Y                       | N                              |

For more details on the interaction check out [Anatomy - Guest Interaction](/anatomy/guest_interaction.md).

## Guest Access Credentials

Depending on the VM guest OS processing options selected (enabled or disabled application-aware
image processing) and on the guest access method, you may need to supply access credentials for the
guest OS, as described in the tables below.

### Windows OS

| Application-Aware Image Processing (AAIP)                                                      | Hypervisor Quiescence   | Veeam via VIX/PSDirect | Veeam via RPC | Disabled (crash-consistent) |
| ---------------------------------------------------------------------------------------------- | ----------------------- | ---------------------- | ------------- | --------------------------- |
| Membership in the local Administrators group                                                   | User account not needed | No                     | Yes           | Not needed                  |
| Enter username as _&lt;servername&gt;\\ Administrator_ [^1] or _&lt;domain&gt;\\Administrator_ | No                      | Yes[^2]                | No            | No                          |
| UAC can be enabled                                                                             | Yes                     | Yes[^3]                | Yes           | Yes                         |
| Guest tools must be installed and up to date                                                   | Yes                     | Yes                    | Yes           | No                          |

### Linux OS

| Linux guest OS processing                    | Hypervisor Quiescence | Veeam via SSH | Disabled (crash-consistent) |
| -------------------------------------------- | --------------------- | ------------- | --------------------------- |
| Root user account                            | No                    | Yes           | No                          |
| User requires `sudo` rights                  | No                    | Yes           | No                          |
| Certificate-based authentication available   | No                    | Yes           | No                          |
| Guest tools must be installed and up to date | Yes                   | Yes           | No                          |

## Sizing

Since guest processing produces very low impact on VM performance, no special considerations on
sizing are required. If you use VSS processing with hypervisor quiescence or Veeam in-guest
processing, you need free space on each drive of the VM for the software VSS snapshot. Please check
Microsoft requirements for more information.

## File Exclusions
Another operation Veeam Backup can do on guest OS level (NTFS only) is excluding certain files or
folders from the backup. Alternatively the job can be configured to include only specified files or
folders in the backup.

This functionality operates very similarly and shares a lot of characteristics with excluding
Windows page file and deleted file blocks. It may help reduce size of the backup files or implement
additional data protection strategies for specific data. Backups for which this option was enabled
remain image-level and hypervisor APIs are used to retrieve VM data. File exclusion feature uses a
combination of NTFS MFT data and guest file system indexes collected by in-guest coordination
process to determine which virtual disk blocks belong to the excluded files and thus should not be
included in the backup.

Full file/folder paths, environment variables or file masks can be used to define exclusions. For
more details on configuring exclusions and its limitations refer to the corresponding
[User Guide section](https://helpcenter.veeam.com/docs/backup/vsphere/guest_file_exclusion.html).

**Note**: Generic file exclusions (defined for high level folders) are most effective. File masks
exclusions require guest file system indexes and generating indexes may put additional stress on
guest VM and will increase backup time. For this reason it is recommended to avoid using file system
masks especially on fileservers with large number (thousands) of small files and use high level
folder exclusions instead. When using include filters, file exclusions are created for everything
else and can take significant time.

---

[^1]: Local administrator accounts other than the built-in Administrator account may not have rights to manage a server remotely, even if remote management is enabled. The remote User Account Control (UAC) `LocalAccountTokenFilterPolicy` registry setting must be configured on the VM guest to allow local accounts of the Administrators group other than the built-in administrator account to remotely manage the server:

    |       |                                                                                                    |
    | ----: | -------------------------------------------------------------------------------------------------- |
    |  Path | `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`                     |
    |   Key | `LocalAccountTokenFilterPolicy`                                                                    |
    |  Type | REG_DWORD (32-bit)                                                                                 |
    | Value | **`1`** = _disable token filter and **allow** remote management by local administrative accounts_  |
    |       | **`0`** (default) = _enable token filter and **do not allow** remote management by local accounts_ |

[^2]: Only this account is able to bypass the UAC prompt for launching processes with administrative privileges. If not applicable, see [^3].

[^3]: When performing application-aware image processing on Windows via VIX, UAC must be entirely disabled, unless the user account is the local administrator account (SID S-...-500).
