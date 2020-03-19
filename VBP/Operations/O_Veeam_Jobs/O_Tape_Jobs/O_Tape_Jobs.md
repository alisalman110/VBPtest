---
title: Operate Tapes
parent: Veeam Tape Jobs
grand_parent: Operate
nav_order: 10
---


#Tapes and guests Backup Modes

Backup jobs can create different backup types of backup file chains on disk depending on the backup mode used. Depending on backup mode, "Backup to Tape" jobs either copies files to tape or synthesize a full backup. The following rules apply:
**When archiving reverse incremental backups**, the behavior varies on the type of media pool used:
- Standard Media Pool: The tape job will always copy the full backup and ignore any rollback files (VRB)
- GFS Media Pool: The tape job will create a full backup from VRB files on specified day(s) as per schedule.
**When archiving forward incremental backups, with active or synthetic full scheduled**, the backup chain on tape will be a copy of the backup chain on disk. The virtual full option in tape job configuration is ignored.
**If you archive forward incremental backups without synthetic or active full enabled, or archive Backup Copy Jobs**, the full files are synthesized from existing restore points on disk. The virtual full backup schedule can be configured on the "Backup to Tape" job. The forever forward incremental chain always keeps on disk one full backup followed by a fixed number of increments. The full backup is constantly rebuilt: as new increments appear, the older ones are injected into the full. The periodic fulls split the forever incremental backup chain into shorter series of files that can be effectively stored to tapes. Each series contains one synthesized full backup and a set of increments. Such series are convenient for restore: you will need to load to the tape device only those tapes that are part of one series. The virtual full does not require additional repository disk space: it is synthesized directly on tape on the fly, when the tape job runs. To build such full backup, Veeam Backup & Replication uses backup files that are already stored on the backup repository. If the primary job produces a forever incremental backup chain or is a backup copy job, Veeam Backup & Replication will periodically create a virtual full backup. You can configure the full backup with the scheduler . The virtual full cannot be switched off; however, it is disabled automatically if the primary job periodically creates active full or synthetic full backups. The virtual full does not depend on the job settings for incremental backups. If you enable the virtual full for the job, it will be created in any case, no matter whether you enable or do not enable incremental backups. If the source backup job contains multiple chains, and the checkbox "Process latest full backup chain only" in advanced job settings is unchecked you will be prompted for a decision when creating a Backup to Tape job. You may choose to either only the last backup chain or all existing restore points.

#Prioritising Tape backups over Primary backups
Sometimes, the primary job may start when the tape job is still running. By default, the primary job has priority. In this case, the tape job terminates with error and no data is written to tape. Select the Prevent this job from being interrupted by primary backup jobs option if you want to give the tape job a higher priority. If this option is selected, the primary job will wait until the tape job finishes. Note that the primary job may start with a significant delay.

#Tape Encryption
Veeam uses hardware encryption if it is provided by the tape device and enabled in Veeam Backup & Replication. Tape library should work in the application-managed encryption mode.

If the hardware based encryption is not supported by the tape device, software based AES-256 encryption is used. Please note software based encryption may cause significant performance degradation, if not natively accelerated by the CPU of the tape server.

Hardware based encryption is typically available for LTO-4 or newer libraries, and while a license is often required, this is usually supported for free by the tape library vendor.

When archiving data, Veeam generates a user key which is stored with data on tape. If you restore data using another Veeam backup server, provide the password or utilize the Password Loss Protection in Enterprise Manager. See the User Guide for more information.

If the hardware encryption option is used, and you archive to tape Veeam backups that are already encrypted on disk, they will be encrypted twice. If you restore such backups with double encryption on the same Veeam backup server they will be decrypted automatically. To decrypt on another Veeam backup server, you will need to enter the two passwords accordingly.

For additional details on tape encryption, see the corresponding section of this guide > Encryption

##Best Practices

- Enable encryption if you plan to store tapes in locations outside of your security domain.
- Consider the risks/benefits of enabling tape job encryption even if the source data is already encrypted and evaluate appropriately the acceptable level of risk.
- Use strong passwords for tape job encryption and develop a policy for changing them regularly (you can use Veeam Backup & Replication password age tracking capability).
- Store passwords in a secure location.
- Obtain Enterprise or a higher level license for Veeam Backup & Replication, configure Veeam Backup Enterprise Manager and connect backup servers to it to enable Password Loss Protection.
- Back up the Veeam Backup Enterprise Manager configuration database and create an image-level backup of the Veeam Backup Enterprise Manager server. If these backups are also encrypted, make sure that passwords are not lost as there will be no Password Loss Protection for these backups.

##Parallel processing for tape jobs

Parallel processing for source chains of one (or more) tape jobs Processing Tape Jobs Simultaneously When you process tape jobs in parallel, the media pool assigns a drive to each running tape job.

The media pool can use the predefined maximum number of drives and process the equal number of tape jobs simultaneously.

For example, if you set 3 drives as the maximum, you can process up to 3 tape jobs at the same time. If you have more jobs running at the same time, they are queued. When one of the jobs finishes and releases its drive, the first queued job takes the drive.

This option is available for backup to tape and file to tape jobs. For example:

- You set the maximum number of drives to 3.
- 4 tape jobs start at the same time. The tape jobs start and jobs A, B and C occupy 3 drives to write data to tape. The Tape job D is queued and waits. When one of the jobs finishes and releases its drive, the Tape job D takes the drive and starts writing data.

##Processing Backup Chains Simultaneously
When you select processing backup chains in parallel, the media pool processes several primary jobs simultaneously. If the primary jobs produce per-VM backups, the media pool processes several per-VM backup chains simultaneously. This option is available for backup to tape jobs only. For example:
You set the maximum number of drives to 3.
Tape job A has 4 primary jobs. Tape job A starts, and occupies 3 drives to process 3 primary jobs. The fourth primary job is queued and waits. When one of the drives is released, the fourth primary job takes the drive and starts writing data.  If another tape job starts, it will be queued and wait until Tape job A finishes

**Note:**  If the media pool is configured to fail over to another library in case all tape drives are busy, only tape jobs can use drives of the next library. You cannot split source backup chains within one job across libraries.

#Tape Restores

##VM Restore from Tape to Infrastructure
Restoring a VM from tape with Veeam Backup & Replication is a lot like restoring a VM from disk. For example, you can choose a desired restore point, select the target location or change the configuration of the restored VM. To restore a VM from tape, you can choose between the following options:

- restore directly to infrastructure
- restore through a staging repository

To choose the needed option, select Restore directly to the infrastructure or Restore through the staging repository at the Backup Repository step of the Full VM Restore wizard.

###Restore Directly to Infrastructure
When you restore VMs from tape directly to the infrastructure, the restore process publishes the VMs to the virtual infrastructure copying the VM data directly from tape. This option is recommended if you want to restore one VM or a small number of VMs from a large backup that contains a lot of VMs. In this case, you do not need to provide a staging repository for a large amount of data most of which is not needed to you at the moment. This option is slow if you restore many VMs. The VMs are restored one by one: this requires a lot of rewinding of tape as tapes do not provide random access to data.

###Restore Through Staging Repository
When you restore VMs from tape through a staging repository, the restore process temporarily copies the whole restore point to a backup repository or a folder on disk. After that Veeam starts a regular VM restore. This option is recommended if you want to restore a lot of VMs from a backup as the disk provides a much faster access to random data blocks than tape.

###Backup Restore from Tape to Repository
This option allows you to copy VM backups from tape to repository. This is helpful if you need some backups on disk for later use, or also for VM guest OS files restore. You can restore full backups or incremental backups to a repository or any location of your choice. The restored backup is registered in the Veeam Backup & Replication console as an imported disk backup so that you can use it for any restore from disk scenario later on. For one restore session at a time, you can choose one restore point available on tape.

##File Restore from Tape
You can restore files and folders that were previously archived with file to tape jobs. Restoring capabilities allows you to restore files to their original location or another server, preserving ownership and access permissions. The file restore process allows you to restore files to any restore point available on tape.

#General Tips
- "Short Erase" all tapes before use with Veeam to avoid any problems cause by data from other backup software.
- Install latest Windows Updates.
- Install latest firmware on library, drives, HBA (verify interoperability).
- Install separate HBAs for tape is recommended, but not required.
- A staging area for backup files is required when restoring from tape. Keep this in mind when sizing backup repositories.
- Tape compression should be disabled for tape jobs, when backup files are already compressed at the backup repository.


<!--- This was last Changed 03-05-17 by PS --->
## Configuring Backup to tape
Before you configure a backup to tape job, complete the following prerequisites:


- You must have Veeam Backup & Replication Enterprise license or higher is installed on the Veeam backup server.

-	You must pre-configure backup job(s) that produce the backup for archiving.

-	The primary backup job must have at least 2 restore points on disk.

-	The primary backup copy job must have at least 4 restore points on disk.

-	You must configure one or more simple media pool with the necessary media set and retention settings.

-	You must load tapes to the tape device and configure the target media pool so that it has access to them. If the media pool has no available tape, the tape job will wait for 72 hours and then terminate.

Mind the following limitations:

- The backup to tape job processes only VBK (full backups) and VIB files (forward incremental backups).  

- If you back up to tape a reverse incremental chain, the tape job will always copy the full backup.  

- Reverse incremental backups (VRB) are skipped from processing.

- Microsoft SQL Server log files (VLB) are skipped from processing.


## Tape Job Encryption

### When to use it?

Tape job encryption should be used any time you want to protect the data stored on tape from unauthorized access by a 3<sup>rd</sup> party. Tapes are commonly transported offsite and thus have a higher chance of being lost and turning up in unexpected places. Encrypting tapes can provide an additional layer of protection if tapes are lost.

If tape jobs are writing already encrypted data to tape (for example, Veeam data from backup jobs that already have encryption enabled), you may find it acceptable to not use tape-level encryption. However, be aware that a user who gets access to the tape will be able to restore the backup files. Although this user will not be able to access the backup data in those files, some valuable information, for example, job names used for backup files, may leak.


