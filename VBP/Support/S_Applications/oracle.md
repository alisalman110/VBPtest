---
title: Oracle
parent: Applications
grand_parent: Supplemental
nav_order: 10
---


# Oracle

Veeam Backup and Replication application-aware processing capability supports backup of virtual or physical machines running Oracle database server.

## Oracle backup workflow

### Oracle on Linux backup workflow

1. Coordination component which will perform all the necessary steps is injected into the guest VM. This component is the same as the one used for Linux application-aware image processing in general.
2. Perform application discovery. This is done using native OS methods, coordination component queries /etc/oraInst.loc and reads inventory.xml which is then compared to /etc/oratab information.
3. Status and version of instance(s) is fetched.
4. Disk group information is retrieved for ASM instances.
5. Log mode is identified, this information will later be used for decisions on how exactly the database has to be processed. Database files, CDB (Oracle 12 only) and current DBID information is retrieved.
7. At this step archive log necessary information was collected and Veeam will start doing actual backup, modifying database state - current archive log is archived and all archive log information is retrieved.
8. PFILE backup is created and archived into the backup metadata.
9. Additional information is collected and recorded (current DBID, SCN, Sequence IDs, database unique name, domain, recovery file destination, basic listener information and current archive log).
10. Coordination component is shut down and then restarted again to finalize the backup: database is put into backup mode and database snapshot is created.

### Oracle on Windows backup workflow

Behavior on Windows depends on the state of VSS writer, Oracle version and database type.

| | VSS enabled | VSS disabled | Pluggable database |
| -- | -- | -- | -- |
| Oracle 11 | Oracle VSS writer is engaged, NOARCHIVELOG databases are shut down and excluded from VSS processing | Same worflow as for Linux | N/A |
| Oracle 12 | Oracle VSS writer is engaged, NOARCHIVELOG databases are shut down and excluded from VSS processing | Same worflow as for Linux | Same workflow as for Linux, VSS writer is skipped |



## Oracle restore and failover workflow

Before the backup the database (in ARCHIVELOG mode only) is put into backup mode, this has to be taken into consideration when performing restore - restoring database server VM is not enough for restoring the service, database has to be put out of backup mode:

    ALTER DATABASE END BACKUP

## Granular item restore

Oracle restore using Veeam Explorer for Oracle uses a combination of executing commands via SSH or RPC depending on the platform, and using the RMAN client. VM disks are mounted to target server using iSCSI (Windows) or FUSE and loop device (Linux). Only database files will be restored, not instance files. Instance files may be recovered through file-level recovery if needed.

Ensure the account used to connect to target/staging server has enough permissions on operating system and database as described in the corresponding section of [User Guide](https://helpcenter.veeam.com/docs/backup/explorers/veo_connection_to_target_server.html?ver=95) or earlier in this guide.

**Note:** When restoring to Linux ensure that account used to connect to restore target server has valid shell.

### Restore workflow

When performing restore Veeam Explorer follows the following steps:

1. Oracle instance/database discovery is performed and information is collected, that includes path validation and disk space availability checks.
2. VM disks are mounted.
3. Target database is shut down and dropped, configuration is cleaned (configuration and temporary instance files).
4. Database is started from the temporary location, if that fails another restore attempt is performed with safe set of parameters.
5. After successful test start from temporary location database is restored to proper location using automatically generated RMAN script.
6. Restore control files are restored after that. Database is updated to specific transaction prior to that in case point in time was selected for restore.
7. Fast Recovery Area parameters are restored and database is upgraded accordingly if restoring 32-bit instance to 64-bit.
8. To finalize restore mounted backup is removed from RMAN repository, restored database is restarted and new DB ID is generated. Remaining bits of the configuration are restored as well - parameter file is restored to proper path along with password file, DBNAME is changed if needed, logs are reset and online logs are recreated.
