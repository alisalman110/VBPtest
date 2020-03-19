---
title: Veeam Tape
parent: Veeam Tape Jobs
grand_parent: Veeam Jobs
nav_order: 10
---


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

### Best Practices

-   Enable encryption if you plan to store tapes in locations outside of your security domain.
-   Consider the risks/benefits of enabling tape job encryption even if the source data is already encrypted and evaluate appropriately the acceptable level of risk.
-   Use strong passwords for tape job encryption and develop a policy for changing them regularly (you can use Veeam Backup & Replication password age tracking capability).
-   Store passwords in a secure location.
-   Obtain Enterprise or a higher level license for Veeam Backup & Replication, configure Veeam Backup Enterprise Manager and connect backup servers to it to enable Password Loss Protection.
-   Back up the Veeam Backup Enterprise Manager configuration database and create an image-level backup of the Veeam Backup Enterprise Manager server. If these backups are also encrypted, make sure that passwords are not lost as there will be no Password Loss Protection for these backups.
