# Veeam Best Practices Guide

* [Anatomy](anatomy/README.md)
  * [vSphere](anatomy/vmware/index.md)
    * [Interaction with vSphere](anatomy/vmware/interaction.md)
    * [Instant VM Recovery](anatomy/vmware/instant_vm_recovery.md)
  * [Hyper-V](anatomy/hyper-v/index.md)
    * [Interaction with Hyper-V](anatomy/hyper-v/interaction.md)
    * [Backup Modes](anatomy/hyper-v/backupmodes.md)
    * [Interaction with Guest OS](anatomy/hyper-v/guestinteraction.md)
    * [Instant VM Recovery](anatomy/hyper-v/instantvmrecovery.md)
  * [Compression and Deduplication](anatomy/compression_and_deduplication.md)
  * [Guest Interaction](anatomy/guestinteraction.md)
  * [Agents](anatomy/agents/agent_mgmt.md)
    * [Agent for Windows](anatomy/agents/vaw/vaw.md)
    * [Agent for Linux](anatomy/agents/val/val.md)
  * [Encryption](anatomy/encryption/index.md)
    * [Network Transport Encryption](anatomy/encryption/network-transport.md)
    * [Job Encryption](anatomy/encryption/job.md)
    * [Repository Encryption](anatomy/encryption/repository.md)
    * [Tape Job Encryption](anatomy/encryption/tape.md)
* [Planning and Preparation](planning/README.md)
* [Deployment and Configuration](deployment/README.md)
  * [DNS Resolution](deployment/dns_resolution.md)
  * [Database](deployment/database.md)
  * [Backup Server](deployment/backup_server.md)
  * [Enterprise Manager](deployment/enterprise_manager.md)
  * [Backup Proxies](deployment/backup_proxies/README.md)
    * [VMware Backup Proxies](deployment/backup_proxies/vmware_proxies.md)
    * [Hyper-V Backup Proxies](deployment/backup_proxies/hyperv_proxies.md)
  * [Backup Repositories](deployment/backup_repositories/index.md)
    * [Block Repositories](deployment/backup_repositories/block.md)
    * [NAS Repositories](deployment/backup_repositories/nas.md)
    * [Object Repositories](deployment/backup_repositories/object.md)
    * [Deduplication Appliances](deployment/backup_repositories/deduplication.md)
    * [Scale-out Backup Repositories](deployment/backup_repositories/scaleout.md)
  * [Tape Servers](deployment/tape_servers.md)
  * [Network Rules](deployment/network_rules.md)
  * [Tuning vSphere](deployment/vsphere.md)
* [Operations](operations/index.md)
  * [Backup Jobs](operations/backup_jobs/README.md)
  * [Backup Copy Jobs](operations/backup_copy_jobs/README.md)
  * [Replication Jobs](operations/replication_jobs/README.md)
  * [Guest Processing](operations/guest_processing.md)
  * [Applications](operations/applications/README.md)
    * [Microsoft Active Directory](operations/applications/active_directory.md)
    * [Microsoft Exchange](operations/applications/exchange.md)
    * [MS SQL Server](operations/applications/mssql_server.md)
    * [Microsoft SharePoint](operations/applications/sharepoint.md)
    * [Oracle](operations/applications/oracle.md)
    * [SAP HANA](operations/applications/sap_hana.md)
    * [MySQL](operations/applications/mysql.md)
    * [IBM Domino](operations/applications/domino.md)
