---
title: Oracle
parent: Applications
grand_parent: Operate
nav_order: 60
---


# Oracle

Veeam Backup and Replication natively supports backup of Oracle database servers and allows for image level and granular Oracle databases restore.

**Note:** 32-bit Oracle instances on 64-bit Linux and Oracle RAC are not supported using regular VM backup jobs or Veeam Agents. However, Veeam Plug-in for Oracle RMAN does support Oracle RAC environments following the considerations [here](https://helpcenter.veeam.com/docs/backup/plugins/oracle_environment_planning.html#rac).

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


## Additional information

* [Oracle backup and restore workflows](../../anatomy/applications/oracle.md)
