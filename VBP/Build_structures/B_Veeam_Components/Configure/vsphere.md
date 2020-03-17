---
title: Tuning vSphere
parent: Configure
grand_parent: Veeam Components
nav_order: 10
---



# Tuning vSphere

Please read the [Anatomy - vSphere - Interaction](/anatomy/vmware/interaction.md) section to
understand more details on the background of these configuration settings.

Do not change the default values unless you have a reason for it.

## Adjust Number of concurrent Snapshots

The default number of concurrently open snapshots per datastore in Veeam Backup & Replication is 4.
This behavior can be changed by creating the following registry key:

- Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
- Key: `MaxSnapshotsPerDatastore`
- Type: `REG_DWORD`
- Default value: `4`

## Mitigate Snapshot Removal Impact

To mitigate the impact of snapshots, consider the following recommendations:

- Upgrade to vSphere 6u1 or newer to use the new Storage vMotion based Snapshot commit processing.
- **Minimize the number of open snapshots per datastore**.
  Multiple open snapshots on the same datastore are sometimes unavoidable, but the cumulative effect
  can be bad. Keep this in mind when designing datastores, deploying VMs and creating backup and
  replication schedules. Leveraging backup by datastore can be useful in this scenario.

- **Consider snapshot impact during job scheduling.**
  When possible, schedule backups and replication job during periods of low activity. Leveraging the
  Backup Window functionality can keep long-running jobs from running during production. See the
  corresponding setting on the **Schedule** tab of the job wizard

- **Use the vStorage APIs for Array Integration (VAAI) where available.**
  VAAI can offer significant benefits:

  - Hardware Lock Assist improves the granularity of locking required during snapshot growth
    operations, as well as other metadata operations, thus lowering the overall SAN overhead when
    snapshots are open.
  - VAAI in vSphere 5.x offers native snapshot offload support and should provide significant
    benefits once vendors release full support.
  - VAAI is sometimes also available as an ESXi plugin from the NFS storage vendor.

- **Design datastores with enough IOPS to support snapshots.**
  Snapshots create additional I/O load and thus require enough I/O headroom to support the added
  load. This is especially important for VMs with moderate to heavy transactional workloads.
  Creating snapshots in VMware vSphere will cause the snapshot files to be placed on the same VMFS
  volumes as the individual VM disks. This means that a large VM, with multiple VMDKs on multiple
  datastores, will spread the snapshot I/O load across those datastores. However, it actually limits
  the ability to design and size a dedicated datastore for snapshots, so this has to be factored in
  the overall design.

  **Note:** This is the default behavior that can be changed, as explained in the VMware Knowledge
  Base: <https://kb.vmware.com/s/article/1002929>

- **Allocate enough space for snapshots.**
  VMware vSphere 5.x puts the snapshot VMDK on the same datastore with the parent VMDK. If a VM has
  virtual disks on multiple datastores, each datastore must have enough space to hold the snapshots
  for their volume. Take into consideration the possibility of running multiple snapshots on a
  single datastore. According to the best practices, it is strongly recommended to have 10% free
  space within a datastore for a general use VM, and at least 20% free space within a datastore for
  a VM with high change rate (SQL server, Exchange server, and others).

  **Note:** This is the default behavior that can be changed, as explained in the VMware Knowledge
  Base: <https://kb.vmware.com/s/article/1002929>

- **Watch for low disk space warnings.**
  Veeam Backup & Replication warns you when there is not enough space for snapshots. The default
  threshold value for production datastores is 10% . Keep in mind that you must increase this value
  significantly if using very large datastores (up to 62 TB). You can increase the warning threshold
  in the backup server options, of the Veeam Backup & Replication UI.
  You can also create a registry key to prevent Veeam Backup & Replication from taking additional
  snapshots if the threshold is breached:

  - Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
  - Key: `BlockSnapshotThreshold`
  - Type: `REG_DWORD`
  - Default value (in GB): `10`

  **Tip:** Use the [Veeam ONE Configuration Assessment Report](https://helpcenter.veeam.com/docs/one/reporter/vm_configuration_assessment.html)
  to detect datastores with less than 10% of free disk space available for snapshot processing.

- **Tune heartbeat thresholds in failover clusters.**
  Some application clustering software can detect snapshot commit processes as failure of the
  cluster member and failover to other cluster members. Coordinate with the application owner and
  increase the cluster heartbeat thresholds. A good example is Exchange DAG heartbeat. For details,
  see [Veeam KB Article 1744](https://www.veeam.com/kb1744).

## Considerations for NFS Datastores

Backup from NFS datastores involves some additional consideration, when the
**virtual appliance (hot-add)** transport mode is used. Hot-add takes priority in the intelligent
load balancer, when Backup from Storage Snapshots or Direct NFS are unavailable.

Datastores formatted with the VMFS file system have native capabilities to determine which cluster
node is the owner of a particular VM, while VMs running on NFS datastores rely on the LCK file that
resides within the VM folder.

During hot-add operations, the host on which the hot-add proxy resides will temporarily take
ownership of the VM by changing the contents of the LCK file. This may cause significant additional
"stuns" to the VM. Under certain circumstances, the VM may even end up being unresponsive.
The issue is recognized by VMware and documented in <https://kb.vmware.com/s/article/2010953>.

**Note**: This issue does not affect Veeam Direct NFS as part of Veeam Direct Storage Access
processing modes and Veeam Backup from Storage Snapshots on NetApp NFS datastores. We highly
recommend you to use one of these 2 backup modes to avoid problems.

In hyperconverged infrastructures (HCI), it is preferred to keep the datamover close the backed
up VM to avoid stressing the storage replication network with backup traffic. If the HCI is
providing storage via the NFS protocol (such as Nutanix), it is possible to force a Direct NFS data
mover on the same host using the following registry key:

- Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
- Key: `EnableSameHostDirectNFSMode`
- Type: `REG_DWORD`
- Default value: `0` _(disabled)_

  **Value = `1`** - when a proxy is available on the same host, Veeam Backup & Replication will
  leverage it. If the proxy is busy, Veeam Backup & Replication will wait for its availability; if
  it becomes unavailable, Veeam Backup & Replication will switch to NBD mode.

If for what ever reason Direct NFS processing can not be used and HotAdd is configured, ensure that
proxies running in the Virtual Appliance mode (Hot-Add) are on the same host as the protected VMs.

To give preference to a backup proxy located on the same host as the VMs, you can create the
following registry key:

- Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
- Key: `EnableSameHostHotAddMode`
- Type: `REG_DWORD`
- Default value: `0` _(disabled)_

  **Value = `1`** – when proxy A is available on the same host, Veeam Backup & Replication will
  leverage it. If proxy A is busy, Veeam Backup & Replication will wait for its availability; if it
  becomes unavailable, another Hot-Add proxy (proxy B) will be used.

  **Value = `2`** - when proxy A is available on the same host, Veeam Backup & Replication will
  leverage it. If proxy A is busy, Veeam Backup & Replication will wait for its availability; if it
  becomes unavailable, Veeam Backup & Replication will switch to NBD mode.

This solution will typically result in deploying a significant number of proxy servers, and may not
be preferred in some environments. For such environments, it is recommended switching to Network
mode (NBD) if Direct NFS backup mode can not be used.

## Storage Latency Control

One question that often arises during the development of a solid availability design is how many
proxy servers should be deployed. There must be a balance between the production infrastructure
performance (as you must avoid overloading production storage), and completing backup jobs in time.

Modern CPUs have many physical cores and can run many tasks simultaneously. The impact of having
many proxy servers reading data blocks from the production storage at a very high throughput may be
negative. With this in mind, many businesses avoided running backup or replication jobs during
business hours to ensure good response time for their end users. Storage Latency Control was
implemented to help avoid this issue.

To understand how storage latency control works and how to configure, please refer to
[Specifying I/O Settings](https://helpcenter.veeam.com/docs/backup/vsphere/options_parallel_processing.html)
section in the Veeam HelpCenter.

The results of enabling Storage Latency Control are very easy to review using the vSphere Client.

![](./media/interaction-vmw-iocontrol-results.png)

### When to Use?

Storage Latency Control provides a smart way to extend backup windows or even eliminate backup
windows, and run data protection operations during production hours.

When Storage Latency Control is enabled, Veeam Backup & Replication measures the storage latency
before processing each VM disk (and also during processing, if **Throttle I/O of existing tasks at**
setting is enabled). Furthermore, if the storage latency for a given datastore is already above the
threshold, committing VM snapshots can be delayed. In some environments, enabling Storage Latency
Control will reduce the overall throughput, as latency increases during the backup window.

However, in most environments having this feature enabled will provide better availability to
production workloads during backup and replication. Thus, if you observe performance issues during
backup and replication, it is recommended to enable Storage Latency Control.

Storage Latency Control is available in Enterprise and Enterprise Plus editions. The Enterprise Plus
customers are offered better granularity, as they can adjust latency thresholds individually for
each datastore. This can be really helpful in infrastructures where some datastores contain VMs with
latency-sensitive applications, while latency thresholds for datastores containing non-critical
systems can be increased to avoid throttling.

## vCenter Server Connection Count

If you attempt to start a large number of parallel Veeam backup jobs (typically, more than 100, with
some thousand VMs in them) leveraging the VMware VADP backup API or if you use Network Transport
mode (NBD) you may face two kinds of limitations:

- Limitation on vCenter SOAP connections
- Limitation on NFC buffer size on the ESXi side

All backup vendors that use VMware VADP implement the VMware VDDK kit in their solutions. This kit
provides standard API calls for the backup vendor, and helps to read and write data. During backup
operations, all vendors have to deal with two types of connections: the VDDK connections to vCenter
Server and ESXi, and vendor’s own connections. The number of VDDK connections may vary for different
VDDK versions.

If you try to back up thousands of VMs in a very short time frame, you can run into the SOAP session
count limitation. This limit is 2000 connections from vSphere 6 but can be increased if required.
For details, see <https://kb.vmware.com/s/article/2004663>.

Veeam’s scheduling component does not keep track of the connection count. For this reason, it is
recommended to periodically check the number of vCenter Server connections within the main backup
window to see if you can possibly run into a bottleneck in future, and increase the limit values on
demand only.

You can also optimize the ESXi network (NBD) performance by increasing the NFC buffer size from
16384 to 32768 MB (or conservatively higher) and reducing the cache flush interval from 30s to 20s.
For details how to do this, see [VMware KB article 2052302](https://kb.vmware.com/s/article/2052302).
After increaing NFC buffer setting, you can increase the following Veeam Registry setting to add
addition Veeam NBD connections:

- Path: `HKLM\SOFTWARE\VeeaM\Veeam Backup and Replication`
- Key: `ViHostConcurrentNfcConnections`
- Type: `REG_DWORD`
- Default value: `7` _(disabled)_

Be careful with this setting. If the buffer vs. NFC Connection ratio is too aggressive, jobs may
fail.

## Veeam Infrastructure cache

A new service in Veeam Backup & Replication v9.5 is **Infrastructure Cache** reflected as the
Windows "Veeam Broker Service". With it, Veeam can cache directly into memory an inventory of the
objects in a vCenter hierarchy. The collection is very efficient as it uses memory and it is limited
to just the data needed by Veeam Backup & Replication.

This cache is stored into memory, so at each restart of the Veeam services its content is lost; this
is not a problem as the initial retrieval of data is done as soon as the Veeam server is restarted.
From here on, Veeam "subscribed" to a specific API available in vSphere, so that it can receive in
"push" mode any change to the environment, without the need anymore to do a full search on the
vCenter hierarchy during every operation.

![](./media/interaction-vmw-infrastructure_cache_1.png)

The most visible effects of this new service are:

- The load against vCenter SOAP connection is heavily reduced, as we have now one single connection
  per Veeam server instead of each job running a new queiry against vCenter;
- Every navigation operation of the vSphere hierarchy is instantaneous;
- The initialisation of every job is almost immediate, as now the Infrastructure Cache service
  creates a copy in memory of its cache dedicated to each job, instead of the Veeam Manager service
  completing a full search against vCenter:

![](./media/interaction-vmw-infrastructure_cache_2.png)

No special memory consideration needs to be done for the Infrastructure Cache, as its requirements
are really low: as an example, the cache for an environment with 12 hosts and 250 VMs is only 120MB,
and this number does not grow linearly since most of the size is fixed even for smaller
environments.

## Security

When connecting Veeam Backup & Replication to the vCenter Server infrastructure, you must supply
credentials that the backup server will use to communicate with the vCenter Server.

The features that Veeam provides, such as backup, restore, replication, and SureBackup, interact
with vSphere at the fundamental level. Certain permissions are required to take snapshots, create
VMs, datastores, and resource groups. Because of this level of interaction, it is generally
recommended that Veeam Backup & Replication uses a restricted account with the permissions that are
required to complete the job.

However, in some environments full administrative permissions are not desirable or permitted. For
those environments, Veeam has identified the minimum permissions required for the various software
functions.

You can also leverage security to restrict the part of the environment that the backup server can
“see”. This can have multiple benefits beyond security in that it lowers the time required to parse
the vCenter Server hierarchy and reduces the memory footprint required to cache this information.
However, care must be taken when attempting to use this level of restriction, as some permissions
must be provided at the very top of the vCenter Server tree. Specifically if you access the vCenter
over a WAN link such scoping can reduce the (management background) WAN traffic.

For a detailed description of accounts, rights and permissions required for Veeam Backup &
Replication operations, see the
["Required Permissions" document](https://www.veeam.com/veeam_backup_9_0_permissions_pg.pdf)
(not changed since V9.0).
