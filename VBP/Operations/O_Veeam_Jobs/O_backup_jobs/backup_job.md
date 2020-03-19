---
title: Backup Job General
parent: Veeam Jobs
grand_parent: Operate
nav_order: 10
---

<!-- Best Practice Section -->

# General

### Exclusions

It is recommended to limit the number of exclusions in backup jobs to reduce complexity.

### Job Chaining

It is generally recommended to utilise the built-in Intelligent Load Balancing (ILB) to handle proxy/repository resources over Job Chaining.


### Load Balancing

When planning job schedules, you should consider balancing the load on source and target disks.


### Binding Jobs to Specific Proxies

In situations where controlling the flow of data more accurately is required automatic proxy selection can be disabled and specific proxies can be specified for a backup job.


### Increasing Deduplication Rate

Per-Job deduplication (default) is recommended when utilizing repository storage that does not provide deduplication or parallel backup streams. Grouping VMs
 running the same operating system or deployed from similar templates into a single job will increase deduplication rate.

Per-VM backup files is recommended with deduplication appliances that support parallel streams. Using Per-VM without deduplication can increase performance due
 to parallelism but will decrease deduplication as the it will apply to the individual VM chain.




### Backup and Backup Copy Job Encryption
It is recommended to follow these guidelines:

-   Enable encryption if you plan to store backups in locations outside of your security domain.
-   While CPU usage for encryption is minimal for most modern processors, some amount of resources will still be consumed. If Veeam backup proxies are already
highly loaded, take it into account prior to enabling job-level encryption.
-   Use strong passwords for job encryption and develop a policy for changing them regularly. Veeam Backup & Replication helps with this, as it tracks passwords’ age.
-   Store passwords in a secure location.
-   Obtain Enterprise or a higher level license for Veeam Backup & Replication, configure Veeam Backup Enterprise Manager and connect backup servers to it to
enable Password Loss Protection.
-   Export a copy of the active keyset from Enterprise Manager.
-   Back up the Veeam Backup Enterprise Manager configuration database and create an image-level backup of the Veeam Backup Enterprise Manager server.
If these backups are also encrypted, make sure that passwords are not lost as there will be no Password Loss Protection for these backups.

Active full backup is required for enabling encryption to take effect if it was disabled for the job previously.

For more information on exporting active keyset from Enterprise Manager see Help Center Link below.

<hr>

<!-- supporting information -->


### Exclusions

While exclusions can be very useful, the virtual infrastructure is dynamic and changes
rapidly. It is quite possible that a VM gets moved to a folder or resource pool that is excluded which makes it unprotected. Monitoring with Veeam ONE is
highly recommended.

Also remember that exclusions have higher priority over inclusions in Veeam Backup & Replication.

### Job Chaining

Chaining backup jobs is convenient in certain circumstances, but should be used with caution. For example, if a job as part of a chain fails or stops responding, the entire job chain delivers poor backup success rate.

Intelligent Load Balancing (ILB) can handle proxy/repository resources by starting multiple jobs in parallel and consequently using all available proxy/repository resources. This allows optimal task scheduling and provides the shortest backup window.

### Load Balancing

Too many jobs accessing the same disk will load the storage significantly; this makes the job run slower or may have a negative impact on the VMs performance. To mitigate this problem,you can utilize Storage Latency Control(or Backup I/O Control) settings.

Veeam has a load balancing method that automatically allocates proxy resources making a choice between all proxies managed by Veeam Backup & Replication
that are available.

### Binding Jobs to Specific Proxies

While configuring a backup job, you can disable the automatic proxy selection. Instead, you can select particular proxies from the list of proxies managed
by Veeam backup server, and assign them to the job. This is a very good way to manage distributed infrastructures; also it helps you to keep performance under control.

For example, you can back up a cluster residing on a multiple blade chassis. In this case, if using virtual proxies, keep the proxies' load well-balanced
 and optimize the network traffic.

Dedicated proxies can be also very helpful if you use a stretched cluster and do not want proxy traffic to cross over inter-switch links.

You can use Proxy Affinity to allow only specific proxies to interact with a given repository.

**Tip:** To optimize load balancing in a distributed environment where backup proxies are located in multiple sites, it is recommended to select all proxies
 from the same site in the corresponding job.

### Increasing Deduplication Rate

Using Per-Job; grouping VMs running the same operating system or deployed from similar templates into a single job will increase deduplication rate.

Using Per-VM without deduplication can increase performance due to parallelism but will decrease deduplication as the it will apply to the individual VM chain.

<hr>

<!-- References -->

## References:

[Exclude Files from Backup Job](https://helpcenter.veeam.com/docs/backup/vsphere/backup_job_excludes_vm.html?ver=100)

[Export Keys from Enterprise Manager](https://helpcenter.veeam.com/docs/backup/em/em_export_import_keys.html?ver=100)

[Binding Jobs to Specific Proxies](https://helpcenter.veeam.com/docs/backup/vsphere/backup_job_storage_vm.html?ver=100)

[Monitoring Protected VMs](https://helpcenter.veeam.com/docs/one/reporter/protected_vms.html?ver=100)

[Job Chaining](https://helpcenter.veeam.com/docs/backup/vsphere/job_schedule.html?ver=100#chain)

[Storage Latency Control](https://helpcenter.veeam.com/docs/backup/vsphere/io_settings.html?ver=100)

[Resource scheduling](https://helpcenter.veeam.com/docs/backup/vsphere/resource_scheduling.html?ver=100).

[Proxy Affinity](https://helpcenter.veeam.com/docs/backup/vsphere/proxy_affinity.html?ver=100)
