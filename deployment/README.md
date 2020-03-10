# CONTENT NOT SURE TO KEEP

## Host and Storage Discovery

To collect information about the virtual infrastructure all managed vCenters and their connected hosts and datastores are periodically rescanned. This rescan process is visible in the **History** tab > **System** section in the Veeam Backup & Replication console. As seen here, the Host discovery process runs every four hours. All the collected information is stored within the configuration database.

The amount of collected information is typically very small however the Host discovery process may take longer or even exceed the default schedule in highly distributed environments[^1]. If hosts or clusters are connected to vCenter over a high-latency link you may consider deploying a Backup server locally on the ROBO, then you  can create a vCenter service account with a limited scope to that particular location in order to reduce the window of the Host discovery process. If the ROBO uses a stand-alone host it is possible to add the host as a managed server directly instead of through vCenter.

**Note:** Avoid adding individual hosts to the backup infrastructure if using shared storage in a vSphere cluster.

![Host and storage discovery](backup_server_data_flow_1.png)

If storage with advanced integration (HPE, NetApp, EMC, Nimble) are added to the **Storage Integration** tab there will additionally be a Storage discovery process periodically rescanning storage hourly. This process checks all snapshots for virtual machine restore points for usage within Veeam Explorer for Storage Snapshots. The Veeam Backup & Replication server itself will not perform the actual scanning of volumes but it will use the management API's of the storage controller to read information about present snapshots. Only proxy servers with required storage paths available will be used for the actual storage rescanning process[^2].

The following table shows the three different scanning workflows:

  | Adding new storage controller                          | Creating new snapshot                                  | Automatic scanning                                     |
  | ------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------ |
  | 1. Collect specific storage information                | 1. Creating new Snapshot                               | 1. Storage Monitor runs in background                  |
  | 2. List of volumes, snapshots, LUNs and NFS exports    | 2. Lists initiators                                    | 2. Detecting new volumes                               |
  | 3. Checking licenses, FC and iSCSI server              | 3. Testing iSCSI, NFS and FC from proxies              | 3. Scanning volumes for snapshots every 10 minutes     |
  | 4. Lists initiators                                    | 4. Searching storage exports in VMware                 | 4. Lists initiators                                    |
  | 5. Searching storage exports in VMware                 | 5. Mapping discovered VMs from datastores to snapshots | 5. Testing iSCSI, NFS and FC from proxies              |
  | 6. Mapping discovered VMs from datastores to snapshots | 6. Export and scan the snapshots with proxies          | 6. Searching storage exports in VMware                 |
  | 7. Export and scan the snapshots with proxies          | 7. Update configuration database                       | 7. Mapping discovered VMs from datastores to snapshots |
  | 8. Update configuration database                       |                                                        | 8. Export and scan the discovered objects with proxies |
  |                                                        |                                                        | 9. Update configuration database                       |

The scan of a storage controller performs, depending on the protocol, several tasks on the storage operating system. Therefore it is recommended to have some performance headroom on the controller. If your controller is already running on >90% CPU utilization, keep in mind that the scan might take significant time to complete.

The scanning interval of 10 minutes and 7 days can be changed with the following registry keys.

-   Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
-   Key: `SanMonitorTimeout`
-   Type: REG_DWORD
-   Default value: 600
-   Defines in seconds how frequent we should monitor SAN infrastructure and run incremental rescan in case of new new instances

-   Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
-   Key: `SanRescan_Periodically_Days`
-   Type: REG_DWORD
-   Default value: 7
-   Defines in days how frequent we should initiate periodic full rescan after Veeam Backup service rescan

Per default Veeam will scan all volumes and LUNs on the storage subsystem. During rescan, each present snapshot produces a snapshot clone, mounts to a proxy server, scans the filesystem, lookup for discovered VMs and unmounts. This is repeated for every present snapshot.

**Example**: A storage system with 50 volumes or LUNs with 10 snapshots for each.
Scanning the entire system means 500 (50x10) mounts and clones are
performed. Depending on the performance of
the storage system and the proxy server, this can take significant time.

To minimize the scan time it is recommended to select the volumes used
by VMware within the setup wizard to avoid the overhead of scanning
unused data volumes.

## Examples

In this section we will outline two examples based on two enterprises with 50 remote/branch offices (ROBO). They have the following common characteristics:

- One vCenter Server in HQ managing all ROBO sites
- Local backup jobs for fast backup and restore performance
- Offsite copies from the ROBO consolidated at HQ for D/R protection

### Example 1: Centralized Job Configuration

IT requires _one_ central management console for the entire backup infrastructure,
administration and job scheduling. The backup administrator can follow these
guidelines:

1.  Install and configure Veeam Backup & Replication in HQ

2.  Add the vCenter Server via the Veeam Backup & Replication console

3.  Add the ROBO backup server as Managed Server in the **Backup Infrastructure** tab

4.  Configure the HQ backup server with the roles Backup Repository and
    optionally WAN accelerator

5.  Configure the ROBO backup server with the roles Backup Proxy, Backup Repository and optionally as WAN accelerator[^5]

6.  Configure one or more Backup Jobs for each ROBO pointing to its local backup repository

7.  At HQ configure one or more Backup Copy Jobs for each ROBO pointing to the backup repository

8.  Install Veeam Backup Console on the ROBO backup server for faster restore via the local Mount Server

**Note:** The remote console installation files are on the same installation media as Veeam Backup & Replication (`\Backup\Shell.x64.msi`)

#### Constraints
Please consider the following constraint:

-   If a WAN link between HQ and a ROBOs fails, no backup jobs will run, as the backup server will not be able to communicate with the remote ESXi hosts via the centralized vCenter Server

-   When performing file-level restore for non-indexed virtual machines at        the ROBO via Veeam Enterprise Manager the restore point will be mounted over the WAN link to HQ for displaying the contents of the restore point. Thus it is recommended to use indexing for such virtual machines



### Example 2: Distributed Job Configuration
IT requires local backup jobs and backup copy jobs (with optional WAN acceleration)
are created at the ROBO. For security considerations, each ROBO is provided with
delegated access to VMware vCenter. Restore capabilities from backup copy jobs
should be configured and managed at HQ as well as delegated restore and license
management for all sites via Veeam Enterprise Manager. The backup administrator
may follow these guidelines:

1.  Install Enterprise Manager at HQ

2.  Install and configure Veeam Backup & Replication on each ROBO

3.  On vCenter Server, create separate service accounts per ROBO with a limited
    scope for displaying only relevant hosts or clusters

4.  At the ROBO, add vCenter Server via the **Backup Infrastructure** tab using
    the scoped service account

5.  _Optional:_ At the ROBO, configure a local WAN accelerator and create or re-use
    an existing WAN accelerator at HQ (please note many-to-one configurations are
    supported)

6.  At the ROBO, add and configure the Repository Server at HQ (please note
    many-to-one configurations are supported)

7.  Configure one or more Backup Jobs at each ROBO pointing to its local
    backup repository

8.  Configure one or more Backup Copy Jobs at each ROBO pointing to the
    centralized backup repository at HQ (use WAN acceleration as needed)

9.  Install Veeam Backup & Replication Console at HQ. When using the remote
    console for connecting to remote instances, it is possible to leverage faster
    file-level or item-level restores at HQ via the console's built-in Mount Server

**Note**: As components are managed by multiple backup servers, always
ensure that the same patch/update/version level is used for the entire
Veeam backup infrastructure.

[^1]: In very large or extremely distributed environments, it is possible to extend the schedule frequency by altering registry key `VolumesDiscover_Periodically_Hours` (REG_DWORD, default: 4)
[^2]: Storage rescan procedure > [Re-Scanning Storage Systems](https://helpcenter.veeam.com/docs/backup/vsphere/storage_rescan.html?ver=95)
[^3]: More information about guest file system indexing in Veeam Helpcenter > [Guest file system indexing](https://helpcenter.veeam.com/docs/backup/vsphere/indexing.html?ver=95)
[^4]: VMware Distributed Resource Scheduler > [VM-Host Affinity Rules](https://pubs.vmware.com/vsphere-60/topic/com.vmware.vsphere.resmgmt.doc/GUID-2FB90EF5-7733-4095-8B66-F10D6C57B820.html)
[^5]: Remember to add sufficient resources if all three roles can run on the remote backup server.