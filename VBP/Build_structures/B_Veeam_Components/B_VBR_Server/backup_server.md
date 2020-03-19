---
title: Backup Server
parent: Veeam Backup & Replication Server
grand_parent: Veeam Components
nav_order: 20
---

# Backup Server

Veeam Backup & Replication is a modular solution that lets you build a scalable availability infrastructure for environments of different sizes and configurations. The Backup Server is the core component. Features & component requirements will affect your decision how you install the backup server e.g. one datacenter or multiple locations.

## Backup Server Placement

The Backup server runs a number of processes, e.g. the Backup Service, Backup Manager services and in some scenarios a Mount Server as well. In this chapter we will evaluate how each of those components are affected by placement of the Backup & Replication server.

By evaluating the roles and understanding the data flow between the services it is possible to optimize overall backup performance and restore throughput significantly.

**Important**: when using Veeam for replicating VMs to a disaster recovery (DR) site, it is recommended to keep the Backup & Replication server in the DR site alongside the replicas. When the backup server is located in the DR site it enables true "1-Click Failover" by being able to start Failover Plans immediately and thus eliminate manual reconfiguration before the failover process can be initiated.

## Deployment Method

You may deploy the Veeam Backup & Replication server as either a physical or virtual server. It will run on any server with Windows Server 2012 R2 or higher installed (64-bit only). Install Veeam Backup & Replication and its components on dedicated machines. Backup infrastructure component roles can be co-installed. The following guidelines may help in deciding which deployment type is the best fit for your environment.

### Virtual deployment

For most cases, virtual is the recommended deployment. As it provides high availability for the backup server component via features like vSphere High Availability, vSphere Fault Tolerance or Hyper-V Failover Clustering. It also provides great flexibility in sizing and scaling as the environment grows.

The VM can also be replicated to a secondary location such as a DR site. If the virtual machine itself should fail or in the event of a datacenter/infrastructure failure, the replicated VM can be powered on. Best practice in a two-site environment is to install the Backup server in the DR site, in the event of a disaster it is already available to start the recovery.

### Physical deployment

In small-medium environments (up to 500 VMs) it is common to see an all-in-one physical server running the Backup & Replication server, backup proxy and backup repository components. This is also referred to as an "Appliance Model" deployment.

An advantage of running the Veeam Backup & Replication server on a physical server is that it runs independently from the virtual platform. This might be an ideal situation when recovering the virtual platform from a disaster. Should the physical server itself fail, there are additional steps to take before reestablishing operations:

  1. Install and update the operating system on a new server
  2. Install Veeam Backup & Replication
  3. Restore the configuration backup

## Configuration backup

It is recommended to store the configuration backup, **using a file copy job**, in a location that is always available to this standby Backup & Replication server.

## Sizing and System Requirements

In this section, we will describe how to configure and size the Veeam backup server.

Sizing with Veeam is cumulative in respect to configurations, if you want to create an all-in-one appliance (Appliance Model) add all the resource requirements together (CPU + Memory) to understand what in total you will need, combining roles such as the proxy and repository merits the same considerations

### Compute and memory requirements

The minimum configuration to start with is **4 Cores and 8 GB RAM**. In Veeam Backup & Replication 9.5 the broker service was introduced which caches the vSphere infrastructure, because of this feature the amount of memory and CPU cycles have been drastically reduced. VMware vSphere infrastructure cache maintains an in-RAM mirror of vSphere infrastructure hierarchy to dramatically accelerate job start up (Building VM list operation) and user interface responsiveness while browsing a virtual infrastructure. This approach removes the load from a vCenter Server, making it more available to perform its core infrastructure management duties, and improves backup success ratio in the environments where jobs would often time out or fail due to an overloaded vCenter Server. The cache is maintained up-to-date with real-time updates via a subscription to vCenter Server infrastructure change events.

Recommended Veeam backup server configuration is **1 CPU core (physical or virtual) and 5 GB RAM per 10 concurrently running jobs**. Concurrent jobs include any running backup or replication jobs as well as any job with a continuous schedule such as backup copy jobs and tape jobs.

For every additional concurrent task above 16, add 512MB RAM to the Backup Management Server.

### Operating system

The Veeam backup server requires Microsoft Windows 2012 R2 or later (64-bit only).

The latest supported version of Windows OS is always recommended as it will also support restoring from virtual machines with ReFS file systems or workloads with Windows Server Deduplication enabled.

For the full list of supported operating systems, please refer to the corresponding [System Requirements](https://helpcenter.veeam.com/docs/backup/vsphere/system_requirements.html#backup_server) section of the Veeam User Guide.

### Installation folder

Default location is `C:\Program Files\Veeam\Backup and Replication`.

Plan for 40 GB. If installing in a virtual machine, thin disks may be used. By default the installer will choose the drive with most available free space for the built in backup repository.

### Log files

Default location is `C:\ProgramData\Veeam\Backup`.

Log file growth will depend on the number and frequency of jobs and the number of instances being protected. Consider that the logging level may also affect the log size, if you need to change the logging level or log file location refer to this Veeam Knowledge Base article: <https://www.veeam.com/kb1825>.

It is recommended to not configure the logging level below 4, as it may complicate troubleshooting. Logging level 6 is very intrusive, and should only be configured for short periods of time when requested by Veeam Support.

Plan for 3 GB log files generated per 100 protected instances, with a 24 hour RPO. For environments with more than 500 protected instances it is recommended to change the default location to a different fast access disk. Many concurrently running jobs may produce a lot of write streams to log files, than can slow down operations for the Veeam Backup Service and Backup Manager processes.

### Veeam Backup Catalog folder

Default location is `C:\VBRCatalog`.

This folder is used if guest indexing in backup jobs is enabled and it is populated when VBR interacts with gues OS's. Indexing requires the following amount of free space in the file system where the VBRCatalog folder is created:

  - 100 MB every million of files for Windows OS's
  - 50 MB every million of files for Linux OS's

Once indexing data have landed, they are processed and compressed. Their size is reduced to 2 MB every million of files in a second time.

To change the default location, refer to this Veeam Knowledge Base article: <https://www.veeam.com/kb1453>

### vPower NFS folder

Default location is `C:\ProgramData\Veeam\Backup\NfsDatastore`.

When booting VMs with Instant VM Recovery or SureBackup, this folder is used by default to store all configuration files and redo logs of the running VM. To offload the changes to a specific production datastore refer to the corresponding page of the Instant VM Recovery wizard.

We recommend installing vPower NFS Services on each Windows-based backup repository. For SMB/CIFS based repositories or deduplication appliances it is recommended to configure vPower NFS on the gateway server. For Linux-based repositories it is recommended to configure vPower NFS on a managed Windows machine as close as possible to the Linux repository (similar to selecting a Gateway Server for SMB/CIFS or storages that use enhanced deduplication technologies).

The vPower NFS server is bound to backup repositories and the folder location is defined per server. To achieve best performance for VMs running off of vPower NFS please configure the fastest possible storage on the backup server or backup repository. To change the folder location  please see the following steps.

  1. In the **Backup Infrastructure**, select the **repository** you wish to change.
  2. Right click the **repository** and go to **properties**
  3. When the wizard opens navigate to the **Mount server** settings
  4. Using the browser buttons locate the new location for your vPower NFS storage
  5. Finish the wizard

It is recommended to reserve at least 10 GB space for this folder. If you plan to start a significant number of VMs or run VMs over a longer period  increase the space accordingly to fit the produced/estimated amount of changes generated by the running VMs (conservative average change rate can be defined as 100 GB per 1 TB VM per 24 hours - or 10%). Additional disk space is consumed when using Quick Migration. See more information [here](https://helpcenter.veeam.com/docs/backup/vsphere/instant_recovery_before_you_begin_vm.html).

**Important!** Make sure vPower NFS is configured correctly on the Veeam backup server itself as it will be used when deploying Virtual Lab for SureBackup or when performing file-level recovery for Linux-based VMs.

For information on folders required for Enterprise Manager, backup proxy and repository servers (backup targets) and WAN accelerators, as well as for recommendations on their sizing please refer to the corresponding sections of this guide.

## Disaster Recovery Optimization

Proper planning dictates that to get 1-Click Failover working it requires that the vSphere clusters in each location are connected to separate vCenter servers. In the event of an outage in the primary datacenter it is only possible for the Backup & Replication server in the DR site to initiate failover if the vCenter server itself is available.

In cases when it is impossible to have multiple vCenter instances across sites (e.g. Metro Cluster or similar active-active configurations), the recommended solution is to use vCenter Server and following these steps in event of a disaster:

 1. Replicate vCenter from primary site to secondary site with low RPO
 2. Configure VMware DRS affinity rules for pinning replica vCenter VM to a specific host
 3. Connect to specified host and manually power on replicated vCenter VM
 4. Verify vCenter availability through Veeam Backup & Replication
 5. Initiate Failover Plans

## Other software

It is strongly recommended that no highly-transactional and business-critical software is deployed on the same machine as the Veeam backup server. This could be (but not limited to) software such as Active Directory, Exchange Server or other intensive production databases on the SQL server instance. If possible it would be preferable to have no other software at all running on the Veeam Backup Server.

It is recommended to follow antivirus exclusion guidelines as explained in [Veeam KB 1999](https://www.veeam.com/kb1999).

If it is not possible to connect to a remote SQL staging server for Veeam Explorers you can install Standard or Enterprise versions of SQL (depending on your licensing) locally for staging databases for item-level restores on the backup server. This installation can also be used to store the Veeam backup database if required as long as sufficient resources are assigned to the host machine, however do not run any instances in production from this installation that may affect the operation of the backups or restore processes. SQL express is included in the distribution but is limited to a 10GB database.

Other software such as Microsoft Outlook (64-bit) for mail export to PST files via Veeam Explorer for Exchange, or a PDF viewer for reading Veeam documentation are considered non-disruptive.

## Installing Veeam Backup & Replication updates

New Veeam releases and updates are installed on the Veeam Enterprise Manager and Veeam backup servers by the setup wizard or by using the unattended installation method (also referred to as “silent installation”). For detailed instructions check the latest release notes.

**Note:** Veeam Backup Enterprise Manager must be updated before updating Veeam backup servers.

After installing updates open the Veeam Backup & Replication management console. The **Update** screen will be displayed and will guide
you through updating distributed components on other Veeam managed servers (like proxy and repository servers, vPower NFS servers, WAN accelerators
and tape servers).

**Note:** As Veeam deploys no agents on the virtual machines, you do not need to update any software (agents) on the VMs.
