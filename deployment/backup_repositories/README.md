# Backup Repositories

## Server-Based Repository: DAS or SAN?

### Direct-Attached Storage (DAS)
This is an easy, fast and lowcost way to use storage. It is a new approach to use microsegmentation instead of monolithic solutions. The DAS approach is in terms of performance a very fast solution. It can be used as a dedicated system to one Cluster or in a Scale-out Backup Repository. DAS is a normal industry standard x64 server with a bunch of disks attached to it. 
-   It is recommended to use a performant RAID controller with local battery cache. Be aware of any RAID overhead when designing a DAS soltuion. Typically RAID 6/60 (depends on the amount of disks) is recommended (IO overhead of factor 6). The Stripe Size should be 256KB or greater.
-   Since a DAS storage can be fully dedicated to backup operations, this type of repository is considered to offer a good balance between “performance” and “cost” factors.
-   A strong benefit of a DAS repository is that it supports the features offered by Veeam Backup & Replication in a very flexible way. In particular, it provides good read and write performance, sufficient for Veeam vPower-based features (such as Instant VM Recovery, SureBackup, and others). As it typically provides good random I/O performance, it will be the optimal solution when using I/O intensive backup modes such as reverse incremental or forever forward incremental (also used in backup copy job).
-   For scalability you can scale vertical (more disks in an enclosure or additional) and horizontal (more servers, if e.g. the network throuput is reached, the SAS channels are saturated, more IOPS are needed for restore reasons) 


**Tip:** When using Microsoft based repositories, use the RAID controller, to build the RAID set and set the Stripe Size there. Don't us any kind of Software or HBA based RAID Level.

| Pros               | Cons                                          |
|:-------------------|:----------------------------------------------|
| Cost               | single Point of Failure is the RAID Controller|
| Performance        |                                               |
| Simplicity         |                                               |
| Microsegmentaion   |

### SAN Storage

This is an advanced and manageable solution that offers the same advantages as DAS, and adds more advantages like higher availability and resiliency.

The volume size and quantity are easily adjustable over time, thus offering a scalable capacity.

**Tip**: You can configure multiple backup repositories on the SAN storage to increase repository throughput to the storage system.

| Pros                   | Cons       |
|:-----------------------|:-----------|
| Reliability            | Complexity |
| Performance            | Cost       |
| Technical capabilities | Monolithic approach     |

## Windows or Linux?
The main difference between Windows and Linux in regards to Veeam repositories is the way they handle NAS shares – this can be summarized as a choice between NFS and SMB. It depends on your IT infrastructure and security what is better to manage and to maintain.