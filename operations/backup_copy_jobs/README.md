# Backup Copy Job

The backup copy job has the following phases:

1. **Pre-job activity** — if enabled, the pre-job scripts are executed at the very beginning of a copy interval.

2. **Health check** — if scheduled, backup file integrity is verified before the next copy is initiated.

3. **Data transfer (synchronization) phase** — during this phase, the backup copy job checks for a new restore point in the source, creates a file for a new restore point at the target and starts copying the state of the latest restore point of each processed VM to the target repository. The data transfer (synchronization) phase starts at specific time configured in the job properties (see [Synchronization Intervals](https://helpcenter.veeam.com/backup/vsphere/backup_copy_sync_interval.html)). You can define any interval needed in minutes, hours or days. Moreover, you can specify the time slot during which data can and cannot be transferred over the network, thus regulating network usage (see [Backup Copy Window](<https://helpcenter.veeam.com/backup/vsphere/backup_copy_window.html>)).

4. **Transform phase** — copy jobs are by nature running in "forever forward incremental" mode, and perform transform operations on the target backup repository accordingly. Additionally, it is possible to schedule health checks or backup file compacting as described in the [Backup Job](./backup_job.md#storage-maintenance) section. The transform phase begins when all VMs are successfully copied to the target, or if the synchronization interval expires.

   **Note:** the transform process itself puts additional pressure on the target repository. In large environments with deduplication storage appliances used as backup repositories or with backup copy jobs processing a large number of VMs or big VMs, the transform process can take a significant amount of time. For non-integrated deduplication appliances, it is recommended to use the "Read entire restore point..." option. This forces the Backup Copy Job to running forward incremental with periodical full backups copied entirely from the source backup repository rather than being synthesized from existing data.

5. **Compact full backups** — if enabled, the recent full backup file is re-created on the same repository, writing all the blocks close to each other as much as possible to reduce fragmentation.

6. **Post-job activity** — if enabled, several post-job activities are executed before the job enters the idle phase, such as post-job scripts and sending e-mail reports.

7. **Idle phase** — for the most time, the backup copy job remains in the *Idle* state, waiting for a new restore point to appear on the source backup repository. When the synchronization interval expires, a new interval starts at step 1.

For more information, refer to the corresponding section of the User Guide > [Backup Copy Job](https://helpcenter.veeam.com/backup/vsphere/backup_copy_job_task.html).

## Job Layout and Object Selection

### Source Object Container

-  **Select from infrastructure**: this selects specific VMs or containers from the virtual infrastructure. The scheduler will look for the most recent restore point containing the VMs within the synchronization interval. The scheduler will look for restore points in all backups, regardless which job generated the restore point. If the restore point is locked (e.g. the backup job creating it is running), the backup copy job waits for the restore point to be unlocked and then start copying the state of the VM restore point according to its defined schedule.
-  **Select from job**: this method of selection is very useful if you have multiple backup jobs protecting the same VMs. In this case, you can bind the backup copy job to a specific job you want to copy. The job container will protect all the VMs in the selected source job(s).
-  **Select from backup**: this method is equivalent to the **Select from infrastructure** method, but allows for selecting specific VMs inside specific backups. This is helpful, when only certain critical VMs should be copied offsite.

### Backup Copy and Tags
As you can select any VM to be copied from multiple backups, you can plan for policy-based configurations. For instance, you may not want to apply GFS retention over some VMs like web servers, DHCP, etc. In this situation, you can use VMware tags to simplify the management of backup copy process. Tags can be easily defined according to the desired backup copy configuration, using VMware vSphere or Veeam ONE Business View to apply tags.

## Initial synchronization

When creating the initial copy to the secondary repository, it is recommended to use backup seeding (see [Creating Seed for Backup Copy Job](https://helpcenter.veeam.com/backup/vsphere/backup_copy_mapping_auxiliary.html))
whenever possible. Especially when transferring large amounts of data over less performant WAN links, the seeding approach can help mitigating initial synchronization issues.

While Backup Copy Jobs were designed for WAN resiliency, the initial copy is more error prone, as it is typically transferring data outside the datacenter over less reliable links (high latency, or packet loss). Another issue that can be solved by seeding is when the full backup is larger than the amount of data that can be transferred in an interval. Even if the interval can be extended to accomodate the initial transfer, this may lead to upload times of even multiple days. Seeding can speed up the initial sync by removing the need for the sync.

The most frequent synchronization issues are described in the User Guide > [Handling Backup Copy Job Issues](https://helpcenter.veeam.com/backup/vsphere/backup_copy_issues.html).


## Additional Options
 
### Restore Point Lookup

By default, after a restart of the job interval (the **Copy every** setting), a backup copy job analyzes the VM list it has to protect, and searches _backwards in time_ for newer restore point states. If the state of the restore point in the target repository is older than the state in the source repository, the new state is transferred.

For example, if the backup job is scheduled to run at 10:20 PM, and the backup copy job uses the default schedule of copying the latest restore point state every day at 10:00 PM, the state copied by the backup copy job is typically one day behind. In the image below, you can see some VMs affected by this behavior.

To change this behavior, it is possible to use the `BackupCopyLookForward` registry key as described below. Reevaluating the example above, using this registry key, the backup copy job will still start searching at 10:00 PM, but will now wait for a new restore point state created after this point in time.

-   Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
-   Key: `BackupCopyLookForward`
-   Type: REG_DWORD
-   Value: 1

The following forum thread provides a very good explanation of the backup copy scheduler and the LookForward registry key > [Veeam Community Forums - Backup Copy Intervals](https://forums.veeam.com/veeam-backup-replication-f2/backup-copy-intervals-t24238.html)

### Backup Copy from Backup Copy

Since v8, it is possible to use a backup copy job as a source for data transfer and to generate another backup copy. For this, select the VMs from infrastructure and specify the backup repository holding the primary backup copy restore points as the source.

### Job Seeding

Usually, a backup copy is used to send data remotely. If it is necessary to send data over a slow link, you can seed the backup copy job by taking the following steps:

1.  Create a "local" backup copy job and target it at a removable device used as a backup repository, or copy the backup files afterwards. Run the created backup copy job to create a full backup set on this device. Note that also the .vbm file has to be moved.
2.  Once the backup copy job is over, delete the local backup copy job from the Veeam console.
3.  Transport the removable device with the created backup files to the destination site.
4.  Copy backup file to the target backup repository.
5.  Import the backup on the target. If already imported, perform a rescan.
6.  Create the final backup copy job on the Veeam console. On the **Target** step of the **Backup copy job** wizard, use the **Map backup** link and select the transported backup — this backup will be used as a “seed”.

If you are using a WAN accelerated transfer, refer to the WAN Accelerator section for proper cache population procedure: <https://helpcenter.veeam.com/backup/vsphere/wan_populate_cache.html>.

**Note:** Only the initial first run of a reverse incremental chain can be used with seeding (but any forward incremental chain can be used). See [kb1856](https://www.veeam.com/kb1856) for more information.
