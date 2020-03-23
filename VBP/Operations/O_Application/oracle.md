---
title: Oracle
parent: Applications
grand_parent: 4-Operate
nav_order: 60
---




# Oracle

Veeam Backup and Replication natively supports backup of Oracle database servers and allows for image level and granular Oracle databases restore.

**Note:** 32-bit Oracle instances on 64-bit Linux and Oracle RAC are not supported using regular VM backup jobs or Veeam Agents. However, Veeam Plug-in for Oracle RMAN does support Oracle RAC environments following the considerations [here](https://helpcenter.veeam.com/docs/backup/plugins/oracle_environment_planning.html#rac).

## Oracle Image-Level backup

## Preparation

Only databases in ARCHIVELOG mode will be backed up online, databases in NOARCHIVELOG mode will be shut down which will cause **database availability disruption**.

Logs are stored temporarily on the guest filesystem before they are shipped for processing. This may cause undesired behavior if there is no enough space available in default location and changing temporary location from default is recommended as per [KB 2093](https://www.veeam.com/kb2093).

When backing up Oracle on Linux, the backup server is used for initiating connections, whereas a Guest Interaction Proxy will be selected for Oracle on Windows.

As restore is integral part of Oracle protection, special attention should be paid to planning Veeam Explorer for Oracle configuration, specifically network connectivity between mount server and staging servers in restricted environments. Ports used for communication between them are listed in the corresponding section of the User Guide (https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95#explorers).

### Permissions

Certain level of access is expected from the user account configured for performing Oracle backup. Refer to the corresponding section of the User Guide for details (https://helpcenter.veeam.com/docs/backup/explorers/veo_connection_to_source_server.html?ver=95).

When processing Linux instances, the same user account specified for application awareness is used to process the Oracle backup. For Windows instances, you may specify two separate accounts.

**Note:** It is not possible to use different accounts to access different Oracle instances running on the same VM, make sure specified credentials can be used to access all instances on a VM in those cases.

#### Windows OS

User account used to connect to a VM should have local administrator privileges on guest VM and read/write access to database files on filesystem level.

In addition this account or separate Oracle account in case it is different should have SYSDBA rights, this can be achieved by adding it to **ora_dba** local group.

#### Linux OS

Root account or account elevated to root should be used to connect to a VM. Automatic adding to **sudoers** can be enabled for the account but note that **sudoers** file entry will not be removed automatically. Persistent **sudoers** file entry with *NOPASSWD: ALL* option can be added manually, for example:

    oraclebackup ALL=(ALL) NOPASSWD: ALL

This account should be included in the **oinstall**[^1] group to access Oracle database files hierarchy, and to **asmadmin** group (where applies).

In addition this account or separate Oracle account in case it is different should have SYSDBA rights, this can be achieved by adding it to **dba** local group.

## Job configuration

Refer to the corresponding section of the User Guide (https://helpcenter.veeam.com/docs/backup/vsphere/replica_vss_transaction_oracle_vm.html?ver=95) for details on configuring Oracle database backup and transaction logs processing.

Avoid using aggressive logs truncation settings for databases protected with Data Guard as it may affect logs synchronization to secondary server. Data Guard should have enough time to transport logs remotely before they are truncated thus generally having "Delete logs older than" option less than 24 hours is not recommended.

## Veeam Plugin for Oracle RMAN

Veeam provides integration with Oracle RMAN to take consistency backup of Oracle environment with RMAN by using SBT_TAPE APIs to write the backup on Veeam Backup & Replication Repository.

## Preparation

## Oracle Enviroment

Veeam performs SQL queries on Oracle database to collect the statistical information about the RMAN job process, based on available resources Oracle can decide to use Temp Tablespace resources, it’s best practice to configure the Temp Tablespace resources to avoid shortage of Temporary tablespace.
You can check the Temp Tablespace size with the below SQL query:

### Oracle Version 12:

SELECT * FROM DBA_TEMP_FREE_SPACE;

### Oracle Version 11:

SELECT   A.tablespace_name tablespace, D.mb_total,

You can also create new temp tablespace, for example the new tablespace for 500 M:

CREATE TEMPORARY TABLESPACE TEMP_NEW TEMPFILE '/DATA/database/ifsprod/temp_01.dbf' SIZE 500m autoextend on next 10m maxsize unlimited;

ALTER DATABASE DEFAULT TEMPORARY TABLESPACE TEMP_NEW;

## Oracle RAC:

Before starting the configuration of Veeam Plugin for Oracle RMAN, It's recommended to check the Oratab file, Oratab file should have entries about all the Oracle RAC nodes, if any node is missing please add that in oratab file.

As a best practice, it’s recommended to install Veeam Pugin for RMAN on all Oracle RAC nodes.

### Oracle Home

It’s recommended to backup Oracle Home along with Oracle RMAN backup, to backup the Oracle Home, you can use Veeam Agent or Veeam backup & replication.

## Veeam Environment:

It’s recommended to use separate backup repository for Oracle RMAN backup, The following resources are required:

Oracle server: 1 CPU core and 200 MB of RAM per currently used channel.
Backup repository server: 1 CPU core and 1 GB of RAM per 5 currently used channels. 


## Additional information

* [Oracle backup and restore workflows](../../anatomy/applications/oracle.md)

