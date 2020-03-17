---
title: Deduplication Appliances
parent: Backup Repositories
grand_parent: Deployment and Configuration
nav_order: 30
---

# Deduplication Appliances

Deduplicated storage systems are often optimized for write operations and can offer rather high
ingest rates. However, any random read I/O may suffer from re-hydration processes required during
restores. For this reason we recommend to use these devices mainly as secondary targets, where
parameters like price per GB are more important than restore performance.

As a storage-agnostic product, Veeam Backup & Replication can use any deduplication appliance as a
repository in different use cases: primary backup repository, backup copy repository, and Virtual
Tape Library (VTL) container.

In general there are two types of deduplication appliances: Integrated and non-integrated.

Integrated appliances do use integrated ways of transporting backup data, like special protols as
`DDBoost` and `Catalyst` or they integrate the Veeam Data Mover service directly in their appliance.

## Deduplication Appliance as a Primary Backup Repository

Unless you are using an integrated appliance, you should configure primary jobs for forward
incremental with active full backups - since jobs with transformation will require block
"de-hydration" and then "re-hydration" on the storage. Such operations require significant time and
I/O.

**Note: _"Re-hydration" is the act of restoring the original blocks in a non-deduplicated form.
During backup-file-transformation the same blocks are read and then written back to the appliance
where they are de-hydrated (deduplicated) again. This two-step process can generate significant load
on the appliance, slowing down operations._**

Also, consider that Instant VM Recovery might not be as fast as expected – unless the deduplication
appliance offers a fast non deduplicated area for the most recent restore points (such as ExaGrid).

The downside of active fulls is the need to transport the entire amount of virtual machines on a
weekly/monthly basis. This can lead to long snapshot commit, so this mode needs to be planned
carefully. It is recommended to limit the use for primary backup jobs to the integrated
deduplication appliances, where synthetic operations can be used.

## Using Deduplication Appliance as a Backup Copy Repository

By default a backup copy job applies transformations to the backup chain. This could lead to the
"de-hydration"/"re-hydration" overhead at the end of the backup copy job cycle, due to synthetic
full or transformation. When using non-integrated appliances, use the option of Active Fulls for
Backup Copy jobs.

If one of the integrated appliances is used, synthetic operations will be performed on the appliance
itself, so they will require minimal additional time and lower I/O.

## Using Deduplication Appliance as a Virtual Tape Library

If a deduplication appliance is used in Virtual Tape Library (VTL) mode, it is required to store the
backup files in a staging area, which is uncompressed. Sending compressed and/or deduplicated backup
files to a VTL will compromise the efficiency of the deduplication appliance.

The repository used for staging should be configured with "Decompress before storing" advanced
option enabled, which ensures previously applied compression at the job level is ignored.

Also, ensure that the appliance meets Veeam tape requirements described in the
[User Guide](https://helpcenter.veeam.com/docs/backup/vsphere/system_requirements.html).

## File-Level Recovery and Veeam Explorers

By design, Veeam Explorers perform a large amount of random read operations on the backup
repository. To optimize for such operations on deduplication devices, following the job and
repository configuration best practices (see below) is paramount. If the recommendations are not
fully implemented, this may lead to significant waiting time when launching file-level recovery or
Veeam Explorers.

To further reduce restore time, it is recommended to enable file-level indexing for backup jobs
located on deduplication repositories. Indexing VMs will remove the waiting time for mounting a
restore point when browsing contents via Enterprise Manager.

## Best Practices

In this section, we will distinguish between integrated and non-integrated deduplication appliances.

Integrated appliances are the ones specifically listed in the
[Backup Repository section](https://helpcenter.veeam.com/docs/backup/hyperv/backup_repository.html)
of the user guide.

If the mentioned integration API is unavailable due to licensing restrictions, or if any other
deduplication appliance is used, the appliance should be considered **non-integrated**.

In order to optimize throughput for deduplication appliances, please use the following configuration
guidelines:

### Job configuration

The following settings are configured in the backup job "Edit" wizard under `Storage > Advanced`.
Options not defined in this table are optional and not related to backup repositories using
deduplication storage.

| Setting                                         | Value                             |
| ----------------------------------------------- | --------------------------------- |
| **Tab: Backup**                                 |
| Backup mode                                     | Incremental                       |
| Create synthetic full backups periodically      | Enabled - if integrated           |
| Transform previous backup chains into rollbacks | Disabled                          |
| Create active full backups periodically         | Enabled - if non-integrated       |
| **Tab: Maintenance**                            |
| Perform backup file health check                | Disabled                          |
| Defragment and compact full backup file         | Disabled                          |
| **Tab: Storage**                                |
| Enable inline data deduplication                | Disabled                          |
| Exclude swap file blocks                        | Enabled                           |
| Exclude deleted file blocks                     | Enabled                           |
| Compression level                               | Optimal                           |
| Storage optimization                            | Local target (16TB+ backup files) |
| Enable backup file encryption                   | Disabled                          |

Hardware assisted encryption is available for EMC DataDomain via DDBoost, but must be configured in 
the integration specific repository configuration. If enabled on the job level data reduction 
efficiency will be significantly degraded.

### Repository configuration

The following settings are configured in the "Edit Repository" wizard under `Repository > Advanced`.

| Setting                                          | Value                                                                                |
| ------------------------------------------------ | ------------------------------------------------------------------------------------ |
| Align backup file data blocks                    | Enabled - only if repository uses fixed block size deduplication (almost never true) |
| Decompress backup data blocks before storing     | Enabled                                                                              |
| This repository is backed by rotated hard drives | Disabled                                                                             |
| Use per-VM backup files                          | Enabled                                                                              |