# Backup Proxies

With backup proxies you can easily scale Veeam backup infrastructure based on the organization demands:

- In a **simple deployment** scenario for smaller environments or POC, the backup proxy is automatically installed on the Veeam backup server as part of the Veeam Backup & Replication installation.
- In **advanced deployments**, the backup proxy role is manually assigned to one or more Windows servers. This approach allows for offloading the Veeam backup server, achieving better performance and reducing the backup window.

Backup proxies can be deployed both in the primary site, where the backup server is located, or in a remote site where additional infrastructure needs being backed up. A proxy server can be installed on any managed Microsoft Windows server added to the backup infrastructure. Depending on whether the proxy server is installed on a physical or virtual machine, different transport modes are available.

A backup proxy handles data traffic between the vSphere or Hyper-V infrastructure and Backup & Replication during backup, replication (at source and target), VM copy, VM migration jobs or VM restore. They are also used to detect and scan snapshots to enable Veeam Explorer for Storage Snapshots features when any supported primary storage is added to the backup server.

Backup proxy operations include the following:

-   Retrieving VM data from production storage
-   In-line source side data deduplication to skip whitespace and redundant blocks reported by vSphere Change Block Tracking (CBT), Veeam File Change Tracking (FCT) for Hyper-V versions from 2008 R2 to 2012 R2 or Resilient Change Tracking (RCT) for Hyper-V 2016.
-   Performing in-line compression and deduplication before sending it to the backup repository (for backup) or another backup proxy (for replication)
-   BitLooker: Applies to VMs running Windows OS and using NTFS. For more information, see the corresponding section of this guide > [Deduplication and Compression - BitLooker](../job_configuration/deduplication_and_compression.md#bitlooker)
- 	AES256 encryption, if enabled.

Technically a backup proxy runs a light-weight transport service that takes a few seconds to deploy. When you add a Windows-based server to Veeam backup management console assigning the proxy role to it, Backup & Replication installs the necessary components, and starts the required services on that server. Any host in a Hyper-V cluster is automatically enabled as proxy server, when it is added to the infrastructure. When a job is started, the backup server manages the dispatching of tasks to proxy servers using its built-in [Intelligent Load Balancer](#intelligent-load-balancing) (ILB).

### Intelligent Load Balancing

To specify the threshold for proxy load an administrator uses the **Max concurrent tasks** proxy setting (one task will be consumed for processing a single VM disk), Backup & Replication uses a unique load balancing algorithm to automatically spread the load across multiple proxies. This feature allows you to increase backup performance, minimize backup time window and optimize data flow.

The default proxy server is configured for 2 simultaneous tasks at installation, whereas subsequently added proxy servers analyze the CPU configuration. The proxy server automatically proposes configuring 1 task per CPU core. During deployment, it is determined which datastores or CSV the proxy can access. This information is stored in the configuration database, and is used at backup time to automatically select the best transport mode depending on the type of connection between the backup proxy and datastore.

After the algorithm identifies all existing backup proxies it distributes tasks via the built-in Real-time Scheduler (RTS):

1.  It discovers the number of tasks being processed at the moment by each proxy and looks for the server with the lowest load and the best connection.
2.  All tasks are added to a "VMs to process" queue. When a proxy task slot becomes available, RTS will automatically assign the next VM disk backup task to it.
3.  Priority goes to the disk that belongs to an already processed VM, after that VMs of already running jobs have next higher priority.

**Tip:** At the repository, which writes the backup data, only one thread is writing to the backup storage _per running job_. If few jobs with a high number of VMs are processed simultaneously, you may experience that these threads cannot fully utilize the available backup storage performance. If throughput per I/O stream is a bottleneck, consider
enabling [per VM backup files](./repository_planning_pervm.md).

**Tip:** Default recommended value is **1** task per core/vCPU, with at least 2 CPUs. To optimize the backup window, you can cautiously oversubscribe the **Max concurrent tasks** count, but monitor CPU and RAM usage carefully.

### Parallel Processing

Veeam Backup & Replication supports parallel processing of VMs/VM disks:

-   It can process multiple VMs within a job simultaneously, increasing data processing rates.
-   If a VM was created with multiple disks, Veeam will process these disks simultaneously to reduce backup time and minimize VMware snapshot lifetime.
- 	RTS gives priority to currently running parallel processes for VM disk backups.

To achieve the best backup window it is recommended to slightly oversubscribe the tasks slots, and start more jobs simultaneously. This allow Veeam to leverage the maximum of the task slots and lead into an optimal backup window.

**Note:** Parallel processing is a global setting that is turned on by default. If you had upgraded from older versions please check and enable this setting.

### Backup Proxy Services and Components

Veeam backup proxy uses the following services and components:

-   **Veeam Installer Service** - A service that is installed and started on the Windows server once it is added to the list of managed servers in the Veeam Backup & Replication console. This service analyses the system, installs and upgrades necessary components and services.
-   **Veeam Transport Service** – A service responsible for deploying and coordinating executable modules that act as "data movers". It performs main job activities on behalf of Veeam Backup & Replication (communicating with VMware Tools, copying VM files, performing data deduplication and compression, and so on).
-   **VeeamAgent.exe process** - a data mover which can be started multiple times (on demand) for each data stream on the proxy. These processes can operate in either read or write mode. When used on a proxy server for backup, they are only performing read operations, while "write" mode is used for writing data on a target backup proxy (replication). Veeam agents in write mode are also used on all repository types, but will not be discussed in this chapter.

# Sizing a Backup Proxy

Getting the right amount of processing power is essential to achieving the RTO/RPO defined by the business. In this section, we will outline the recommendations to follow for appropriate sizing.

## Processing Resources

As described above, you may define the max concurrent tasks value in the
backup proxy settings. It is best practices to plan for 1 physical core or 1 vCPU
and 2 GB of RAM for each of the tasks. A task processes 1 VM disk at a time and CPU/RAM
resources are used for inline data deduplication, compression, encryption and other features that are
running on the proxy itself.

In the User Guide it is stated that proxy servers require 2 GB RAM + 500 MB per task.
Please consider these values as minimum requirements. Using the above
mentioned recommendations allow for growth and additional inline processing features
or other special job settings that increase RAM consumption.

If the proxy is used for other roles like Gateway Server for SMB shares, EMC
DataDomain DDBoost, HPE StoreOnce Catalyst or if you run the backup repository
on the server, remember stacking system requirements for all the different components.
Please see related chapters for each components for further details.

**Tip:** Doubling the proxy server task count will - in general - reduce the backup window by 2x.

## Calculating required proxy tasks

Depending on the infrastructure and source storage performance, these numbers
may turn out being too conservative. We recommend to performing a POC to
examine the specific numbers for the environment.

$$ D = \text{Source data in MB} $$

$$ W = \text{Backup window in seconds} $$

$$ T = \text{Throughput in MB/s} = \frac{D}{W} $$

$$ CR = \text{Change rate} $$

$$ CF = \text{Cores required for full backup} = \frac{T}{100} $$

$$ CI = \text{Cores required for incremental backup} = \frac{T \cdot CR}{25} $$

### Example

Our sample infrastructure has the following characteristics:

- 1,000 VMs
- 100 TB of consumed storage
- 8 hours backup window
- 10% change rate

By inserting these numbers into the equations above, we get the following
results.

$$ D = 100\text{ TB} \cdot 1024 \cdot 1024 = 104\,857\,600 \text { MB}$$

$$ W = 8\text{ hours} \cdot 3600 \text{ seconds} = 28\,800 \text{ seconds}$$

$$ T = \frac{104857600}{28800} = 3\,641 \text{ MB/s}$$

We use the average throughput to predict how many cores are required
to meet the defined SLA.

$$ CF = \frac{T}{100} \approx 36 \text{ cores} $$

The equation is modified to account for decreased performance for incremental
backups in the following result:

$$ CI = \frac{T \cdot CR}{25} \approx 14 \text{ cores} $$

As seen above, incremental backups typically have lower compute requirements,
on the proxy servers.

Considering each task consumes up to 2 GB RAM, we get the following result:

**36 cores and 72 GB RAM**

- For a physical server, it is recommended to install dual CPUs with 10 cores each.
  2 physical servers are required.
- For virtual proxy servers, it is recommended to configure multiple proxies
  with maximum 8 vCPUs to avoid co-stop scheduling issues. 5 virtual proxy
  servers are required.

If we instead size only for incremental backups rather than
full backups, we can predict alternative full backup window with less compute:

$$ WS = \frac{104857600}{14 \cdot 100} $$

$$ W = \frac{WS}{3600} \approx 21\text{ hours} $$

If the business can accept this increased backup window for periodical full
backups, it is possible to lower the compute requirement by more than 2x and
get the following result:

**14 cores and 28 GB RAM**

- For a physical server, it is recommended to install dual CPUs with 10 cores each.
  1 physical server is required.
- For virtual proxy servers, it is recommended to configure multiple proxies
  with maximum 8 vCPUs to avoid co-stop scheduling issues. 2 virtual proxy
  servers are required.

If you need to achieve a 2x smaller backup window (4 hours), then you may double
the resources - 2x the amount of compute power (split across multiple servers).

The same rule applies if the change rate is 2x higher (20% change rate).
To process a 2x increase in amount of changed data, it is also required to double
the proxy resources.

**Note:** Performance largely depends on the underlying storage
and network infrastructure.

Required processing resources may seem too high if compared with
traditional agent-based solutions. However, consider that instead of
using all VMs as processing power for all backup operations (including
data transport, source deduplication and compression), Veeam Backup &
Replication uses its proxy and repository resources to offload the virtual
infrastructure. Overall, required CPU and RAM resources utilized by backup
and replication jobs are typically below 5% (and in many cases below 3%) of
all virtualization resources.

## How many VMs per job?

* For per job backup files: 30 VMs per job
* For per VM backup files: 300 VMs per job

Consider that some tasks within a job are still
sequential processes. For example, a merge process writing the oldest
incremental file into the full backup file is started after the last VM finishes
backup processing. If you split the VMs into multiple jobs these background
processes are parallelized and overall backup window can be lower.
Be as well careful with big jobs when you use Storage Snapshots at Backup
from Storage Snapshots. Guest processing and Scheduling of jobs that contain
multiple snapshots can lead into difficult scheduling situation and jobs
spending time waiting for (free) resources. A good size for jobs that
write to per VM chain enabled repositories is 50-200 VMs per Job.

Also, remember that the number of concurrently running backup jobs should not exceed 100. Veeam can handle more, but
a “sweet spot” for database load, load balancing and overall processing
is about 80-100 concurrently running jobs.

## How Many Tasks per Proxy?

Typically, in a virtual environment, proxy servers use 4, 6 or 8 vCPUs,
while in physical environments you can use a server with a single quad
core CPU for small sites, while more powerful systems (dual 10-16 core CPU)
are typically deployed at the main datacenter with the Direct SAN Access
processing mode.

**Note**: Parallel processing may also be limited by max concurrent
tasks at the repository level.

So, in a virtual-only environment you will have slightly more proxies
with a smaller proxy task slot count, while in a physical infrastructure with
good storage connection you will have a very high parallel proxy task
count per proxy.

The “sweet spot” in a physical environment is about 20 processing tasks on a 2x10 Core CPU proxy with 48GB RAM and two 16 Gbps FC cards for read, plus one or two 10GbE network cards.

Depending on the primary storage system and backup target storage
system, any of the following methods can be recommended to reach the
best backup performance:

-   Running fewer proxy tasks with a higher throughput per current proxy
    task

-   Running higher proxy task count with less throughput per task

As performance depends on multiple factors like storage load,
connection, firmware level, raid configuration, access methods and
more, it is recommended to do a Proof of Concept to define optimal
configuration and the best possible processing mode.

## Considerations and Limitations

Remember that several factors can negatively affect backup resource
consumption and speed:

-   **Compression level** - It is not recommended to set it to *"High"*
    (as it needs 2 CPU Cores per proxy task) or to *Extreme* (which
    needs a lot of CPU power but provides only 2-10% additional
    space saving). However, if you have a lot of free CPU ressources
    during the backup time window, you can consider to use *"High"* compression
    mode.

-   **Block Size** - The smaller the block size, the more RAM is needed for deduplication. For example, you will see a increase in RAM consumption when using *"LAN"* mode compared to Local target, and even higher RAM load (2-4 times) when using *"WAN"*. Best practice for most environments is to use default job settings (*"Local"* for backup jobs and *"LAN"* for replication jobs) where another is not mentioned in the documentation or this guide for specific cases.

-   **Antivirus** - see the corresponding [KB](https://www.veeam.com/kb1999) for the complete list of paths that need to be excluded from antivirus scanning

-   **3rd party applications** – it is not recommended to use an
    application server as a backup proxy.
