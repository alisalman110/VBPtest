---
title: Replication Jobs
parent: Veeam Replication Jobs
grand_parent: Veeam Jobs
nav_order: 20
---

# Replication Job

- Place a Veeam backup server on the replica target site
- Plan failover carefully, in particular, DNS and Authentication on both sites
- Replication bandwidth estimations can be made using Veeam ONE
- You can manage network traffic by applying traffic throttling rules or limiting the number of data transfer connections.
- WAN Accelerators (Enterprise Plus) can help on low-bandwidth sites

<hr>

### Offsite Replication

It is recommended to place a Veeam backup server on the replica target side so that it can perform a failover when the source side is down. When planning off-site replication, consider advanced possibilities:
- Replica seeding
- Replica mapping
- WAN acceleration.

These mechanisms reduce the amount of replication traffic while network mapping and re-IP streamline replica configuration.

Plan for possible failover carefully. DNS and possibly authentication services (Active Directory, for example, or DHCP server if some replicated VMs do not use static addresses) should be implemented redundant across both sides. vCenter Server (and vCD) infrastructure should be as well considered for the failover scenario.

In most cases, Veeam does not need a vCenter Server for replica target processing. It can be best practice to add the ESXi hosts from the replica target side (only) directly to Veeam Backup & Replication as managed servers and to perform replication without vCenter Server on the target side. In this scenario, a failover can be performed from the Veeam console without a working vCenter Server itself (for example to failover the vCenter Server virtual machine).

Replication bandwidth estimation has always been a challenge because it depends on multiple factors such as the number and size of VMs, change rate (at least daily, per RPO cycle is ideal), RPO target, replication window. Full information about these factors, however, is rarely at hand. You may try to set up a backup job having the same settings as the replication job and test the bandwidth (as the backup job will transfer the same amount of data as the replication job). **Veeam ONE** (specifically Infrastructure Assessment report packs) may help with estimating change rates and collecting other information about the infrastructure.

Also, when replicating VMs to a remote DR site, you can manage network traffic by applying traffic throttling rules or limiting the number of data transfer connections.


### Replication from Backups

When using replication from backup, the target VM is updated using data coming from the backup files created by a backup or backup copy job.

In some circumstances, you can get a better RTO with an RPO greater or equal to 24 hours, using replicas from backup. A common example beside the usage of proactive VM restores is a remote office infrastructure, where the link between the remote site and the headquarters provides limited capacity.

In this case, the data communication link should be mostly used for the critical VM replicas synchronization with a challenging RPO. Now, assuming that a backup copy job runs for all VMs every night, some non-critical VMs can be replicated from the daily backup file. This requires only one VM snapshot and only one data transfer.


### Backup from Replica

It may appear an effective solution to create a VM backup from its offsite replica (for example, as a way to offload a production infrastructure); however, this design is not at all valid because of VMware limitations concerning CBT (you cannot use CBT if the VM was never started). There is a very well documented forum thread about this subject: [Link](https://forums.veeam.com/vmware-vsphere-f24/backup-the-replicated-vms-t3703-90.html).

#### Other Notes

- Replication of encrypted VMs is supported. Replication of encrypted VMs is NOT supported when the target is Veeam Cloud Connect.
- Veeam Backup and Replication supports replicating VMs residing on VVOLs, but VVOLs are not supported as replication target datastore.
- WAN accelerators (9.5) use single thread operators as opposed to Veeam's typical replication which is multi-threaded. Careful consideration needs to be had on how this will affect the speed of the replicas.



<hr>

### Links

[Veeam Replication](https://helpcenter.veeam.com/docs/backup/vsphere/encrypted_vms_backup.html?ver=100)

[Network Traffic Throttling](https://helpcenter.veeam.com/docs/backup/vsphere/network_rules.html?ver=100)

[Virtual Appliance Mode](https://helpcenter.veeam.com/docs/backup/vsphere/virtual_appliance.html?ver=100)

[Network Mode](https://helpcenter.veeam.com/docs/backup/vsphere/network_mode.html?ver=100)

[WAN Acceleration](https://helpcenter.veeam.com/docs/backup/vsphere/wan_acceleration.html?ver=100)
