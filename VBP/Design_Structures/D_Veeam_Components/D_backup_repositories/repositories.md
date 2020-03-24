---
title: Veeam Backup Repositories
parent: Veeam Backup Repositories
grand_parent: Veeam Components Designs
nav_order: 10
---

<!-- Taken from the B_backup_repositories > repositories.md -->

# Veeam Backup Repositories

Introduction and see link at bottom of page

## Repository Placement

## Repository OS

## Repository Sizing

Being storage-agnostic, Veeam Backup & Replication supports a wide range of repository types, each
offering its own set of specific capabilities. So when deciding on repository storage, you might
consider the following:

- Capacity
- Write performance
- Read performance
- Data density
- Security
- Backup file utilization

As a basic guideline, a repository should be highly resilient, since it is hosting customer data.
It also needs to be scalable, allowing the backup to grow as needed.

Organization policies may require different storage types for backups with different retention.
In such scenarios, you may configure two backup repositories:

- A high-performance repository hosting several recent retention points for instant restores and
  other quick operations
- A repository with more capacity, but using a cheaper and slower storage, storing long-term
  retention points

You can consume both layers by setting up a backup copy job from the first to the second repository,
or leverage Scale-out Backup Repository, if licensed.

## Per-VM Backup Files

It is possible to write one backup file chain per each VM on a repository, compared to the regular
chain holding data for all the VMs of a given job. This option greatly eases job management,
allowing to create jobs containing much more VMs than jobs with single chains, and also enhances
performance thanks to more simultaneous write streams towards a repository, even when running a
single job.

In addition to optimizing write performance with additional streams to multiple files, there are
other positive side effects as well. When using the forward incremental forever backup mode, you may
experience improved merge performance. When backup file compacting is enabled, per VM backup
files require less free space: instead of requiring sufficient space to temporarily accommodate an
additional entire full backup file, only free space equivalent to the largest backup file in the job
is required. Parallel processing to tape will also have increased performance, as multiple files can
be written to separate tape devices simultaneously.

![Per VM backup files](./media/per_vm_backup_files.png)

Per VM backup files is an advanced option available for backup repositories, and it is disabled by
default for new backup repositories. When enabling the option on an repository an active full backup
is necessary on to existing jobs to apply the setting.

Using per-VM backup file will negatively impact repository space usage since Veeam deduplication
is file based. If backup jobs have been created while grouping similar guests to optimize
deduplication and an Active Full is used, per VM Backup chain might require additional repository
space.

** NOTE: _In Scale-Out Backup Repositories, Per-VM backup files option is ENABLED by default_ **

## Concurrent Tasks
Start with configuring one concurrent task per CPU core and adjust based on load of the server,
storage and network.

Dependent on the used storage too many write threads to a storage might be counter productive. For
example, a low range NAS storage will probably not react very well to a high amount of parallel
write processes created by per VM backup files. To limit these effects its better to limit the
concurrent tasks in this case. On the other hand a high-end deduplication appliance might have a
limit on a single write thread but can handle a lot of parallel tasks very well and thus profits
from the use of per-VM backup files and enough concurrent repository tasks.

** Note: _Consider tasks for read operations on backup repositories (like backup copy jobs)._ **

## Physical or Virtual?

You can use a virtual machine as a repository or gateway server, however, keep in mind that the
storage, the associated transport media and the network will be heavily occupied.

If you are using a SAN storage, it can be accessed through software iSCSI initiators, or directly
(as a VMDK or RDM mounted to the Repository VM).

Best practice is to avoid using the same storage technology that is used for the virtualized
infrastructure, as the loss of this single system would lead to the loss of both copies of the data,
the production one and the backups.

In general we recommend whenever possible to use physical machines as repositories, in order to
maximize performance and have a clear separation between the production environment that needs to be
protected and the backup storage.

### General Guidelines for Virtual Repositories

- Each vCPU core should have no more than 2 concurrent tasks loaded into it
- For each vCPU, you need to count 4 GB of memory
- With bigger machines, also connectivity limits come into play, so we recommend to build many
  smaller repositories

Suppose you have a VM with 8 vCPU. You need 32Gb of memory, and this repository could be able to
handle up to 16 concurrent tasks (we can go higher, but that’s a good starting point). Now, in
general we say that one task doing incremental backups runs at about 25 MB/s, that is 200 Mbit/s.
If the hypervisor has a 10Gbit/s link for backup repository traffic and its divided by 200 Mbit/s,
that gives you a bottleneck at around 50 tasks, which would be 25 vCPUs. So, it is not recommended
to go above this size or you may end up to overload the network.

Also take into account that multiple virtual repos may be running over the same physical ESXi and
it's links, so you also need to consider using anti-affinity rules to keep those repos away from
each other.

## Backup File Size
Best practice is to keep the backup chain size (sum of a single full and linked incrementals) under
10 TB (~16TB of source data).

Remember that very big objects can become hardly manageable. Since Veeam allows a backup chain to be
moved from one repository to another with nothing more than a copy/paste operation of the files
themselves, it is recommended to keep the file sizes manageable. This will allow for a smooth,
simple and effortless repository storage migration and better storage-use distribution in SOBRs.

Per-VM backup files do help here, as the size of the chain is only dependent on the size of single
VMs.

---

## References

- [Veeam Help Center - Per-VM Backup Files](https://helpcenter.veeam.com/docs/backup/vsphere/per_vm_backup_files.html)
