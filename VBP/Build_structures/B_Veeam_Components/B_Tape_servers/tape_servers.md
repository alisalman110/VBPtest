---
title: Build_Tape
parent: Veeam Tape Services
grand_parent: Veeam Components
nav_order: 80
---

#Tape Configuration Requirements
Before you configure a backup to tape job, complete the following prerequisites:

- You must have Veeam Backup & Replication Enterprise license or higher is installed on the Veeam backup server.
- You must pre-configure backup job(s) that produce the backup for archiving.
- The primary backup job must have at least 2 restore points on disk.
- The primary backup copy job must have at least 4 restore points on disk.
- You must configure one or more simple media pool with the necessary media set and retention settings.
- You must load tapes to the tape device and configure the target media pool so that it has access to them. If the media pool has no available tape, the tape job will wait for 72 hours and then terminate.

Mind the following limitations:

- The backup to tape job processes only VBK (full backups) and VIB files (forward incremental backups).
- If you back up to tape a reverse incremental chain, the tape job will always copy the full backup.
- Reverse incremental backups (VRB) are skipped from processing.
- Microsoft SQL Server log files (VLB) are skipped from processing.

#Tape Media management

##Media Information

Veeam Backup Database Veeam Backup & Replication catalogues information of all archived data and stores this information in the Veeam backup database. The registered tapes stay in the database until you remove the information about them. You can always view details for each tape, for example, information about backups written to it, even if the tape is not inserted in the library. The catalogue lets quickly detect location of the required items on tape. The catalogue correlates the archived files and the restore points to the names of the corresponding tapes. Both online or offline, and the names of the media sets within were the data was written.
When you start restore, Veeam Backup & Replication prompts for the tapes you need to bring online. As a result, you can restore data from tape much quicker. Veeam Backup & Replication uses the following catalogues for storing the tape-related data:
- Tape Catalogue stores information about files/folders archived to tape media with file to tape jobs, as well as backup files produced by backup to tape jobs. The content of the Tape catalogue can be examined in the Files view.
- Backup catalogue stores information about VMs whose backups are archived to tape media with backup to tape jobs. The content of the Backup catalogue can be examined under the Backups > Tape node in the Backup & Replication view

##Media Pool

A media pool simply defines a group of tapes managed by Veeam Veeam Backup & Replication. There are three types of media pools:
**Service media pools.** Created and managed automatically. It is not possible to modify their settings. They contains:
- Empty media starts out in the **Free pool** indicating it’s available for use in other pools.
- Unknown media will be placed to the **Unrecognized pool** so that it is not overwritten.
- After inventory or cataloging, media with existing data is placed into the **Imported pool**. Review the contents and place such media into the Free pool for overwrite or leave in Imported pool to keep the data.
- Exhausted or broken tapes are placed into the **Retired pool** and are not used further.

**Media pools** are groups of media to which backup data can be written.
- You can create as many custom media pools as needed.
- Media can be assigned to a pool manually, or configured to be automatically assigned from the free pool.
- Configure each pool settings according to the purpose of the pool, such as the overwrite protection period that is applied to all media within the pool.
- Since v9 a (Custom) Tape Pool can be spanned over multiple tape libraries. The idea is to use the capacity and drives of multiple tape systems together and to failover to another tape library in case one library goes offline.

**GFS media pools** are used to store weekly, monthly, quarterly and yearly backups on tape.
- You can create as many GFS tape pools as needed.
- Media can be assigned to a pool manually, or configured to be automatically assigned from the free pool. As well optional can define specific tapes for specific media sets (for example yearly backups).
- Configure each pool settings according to the purpose of the pool, such as the overwrite protection period that is applied to all media within the pool.

##Media Set
A media set is a subset of a media pool that contains at least one backup. A new media set can be created for every backup, or on a time based schedule (i.e. weekly). It is also possible to reuse the same media set forever. When a media set contains at least one full backup, it is a self-sufficient restore point. It means that if you have all tapes from the media set at hand, you can be sure that restore will be successful.

##Media vault
A media vault is used to organize offline media. For example, you have a service organization that transports the tapes to a safe at a bunker. You can name the vault accordingly and add some useful information in the description (phone number, place, etc.). When you need to transport physical tapes to the safe, add these tapes to the vault manually or set automatic export of offline tapes to a vault in the tape jobs or media pools properties.

##Automated Drive Cleaning
You can instruct Veeam Backup & Replication to automatically clean the tape library drives. Assigning the automated cleaning to Veeam Backup & Replication prevents possible overlapping of cleaning tasks and tape jobs. Such overlapping may cause tape jobs failures.

If you enable the automated drive cleaning option in Veeam Backup & Replication, make sure that you disabled the drive cleaning tasks on your tape library device.

Veeam Backup & Replication cleans the drives at the beginning of backup to tape jobs or file to tape job run. The cleaning is not performed during other tape operations such as, for example, cataloging or export. To clean the drives automatically, Veeam Backup & Replication performs the following actions:
- The tape library alerts Veeam Backup & Replication on a drive that requires cleaning.
- Veeam Backup & Replication waits for a tape job to start.
- When the tape job locks necessary drives for writing data, Veeam Backup & Replication checks which of them requires cleaning.
- Veeam Backup & Replication ejects the tape from the drive, inserts a cleaning tape and performs the cleaning.
- Veeam Backup & Replication ejects the cleaning tape and inserts the tape that was reserved for the tape job.
- The tape job writes the data on tape.
- The cleaning process usually takes several minutes.
- The cleaning tapes are located in the Unrecognized media pool. The worn-out cleaning tapes are moved to the Retired media pool automatically.

If a tape job locks multiple drives simultaneously for parallel processing, and one or more drives require cleaning, all drives wait until the cleaning is finished. After cleaning, all drives start writing simultaneously.

The automated drive cleaning does not affect creation of media sets.

**Limitations for Automated Drive Cleaning:** You cannot enable the automated drive cleaning on standalone tape drives. You cannot start the drive cleaning manually with Veeam Backup & Replication. The drive cleaning is fully automated.

##Using 3rd party tape software
As Veeam Backup & Replication tracks and orchestrates all backups written to tape, Veeam recommends using the built-in Veeam tape features (Backups to Tape and Files to Tape jobs).

However, in some situations you may want to use an existing library with non-LTO tapes, or you need to integrate Veeam Backup & Replication into an existing backup-to-tape software. Veeam backup files contain all information needed for restore (e.g. deduplication information, VM metadata, etc.), and you can use the existing backup-to-tape solution to bring the Veeam backup files on tape. This approach can also support enterprise customer "Segregation of duty" demands as two complete different teams can handle backups and tape backups. No single person can delete by mistake or on purpose the primary and tape chain. Before having two backup solutions co-exist on the same server, please verify they do not conflict each other.

# Tape job source selection

##Backup Repositories as Source
When you add a repository as source to tape job, the tape job constantly scans the selected repository (or repositories) and writes the newly created backups to tape. The tape job monitors the selected repository in a background mode. You can set explicit backup windows for the tape job. In this case, the tape job will start on the set time and archive all new restore points that were created in period since the last job run. If you create or remove backup jobs that use this repository, or if you change the configuration of such backup jobs, you do not need to reconfigure the tape job that archives the repository. Mixed Jobs To one tape job, you can link an unlimited number of sources. You can mix primary jobs of different type: backup and backup copy, and of different platform (VMware, Hyper-V, Windows Agent or Linux Agent). You can add jobs and repositories as source to the same tape job. Important! The tape job looks only for the Veeam backups that are produced by backup jobs running on your console. Other files will be skipped. Note that to back up files, you need to configure file to tape job.

##Linking Primary Jobs
You can add primary jobs to tape jobs at any moment: when you create a tape job, or later. Adding primary jobs is not obligatory when you create a tape job: you can create an "empty" job and use it as a secondary destination target. When you link jobs, the tape job processes them in the same way as the jobs added with the Tape Job Wizard. For more information, see Linking Backup Jobs to Backup to Tape Jobs.