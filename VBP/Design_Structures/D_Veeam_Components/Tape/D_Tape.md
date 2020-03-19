#Tape Deployments
To connect tape devices to Veeam Backup & Replication, you need to deploy a tape server. The Tape Server is a Veeam Role that direct connect to tape libraries and to the Veeam backup server and manage traffic between tape devices and Veeam backup server. The connected tape devices are recognized by the Veeam Backup & Replication automatically.
The Data Movers run on tape servers and other components of backup infrastructure. They receive tasks from the Veeam backup server and communicate to each other to transfer the data. The Data Movers are light-weight services that take a few seconds to deploy. Deployment is fully automated: when you assign a tape server role to a server, Veeam Backup & Replication installs the necessary components on this server and starts the required services.

[Typical tape deployment](./media/tape-device_deployment.png)

##Data Block Size
Tape Drives use hardware dependent block sizes to read/write tape data. General the drives support a range of block sizes and report this range to Veeam Backup & Replication. If you use a tape library with multiple drives or a number of standalone drives, Veeam Backup & Replication uses a unified block size to write data to tapes. Veeam Backup & Replication collects the block size ranges reported by each drive, compares them and detects a range of block sizes that can be supported by all drives. This range is additionally limited by storage controllers settings used in your infrastructure. From this range, Veeam Backup & Replication supports only values divisible by 1024. You can check the resulting range of block sizes supported by Veeam Backup & Replication for a particular drive in the Drives properties. For details, see Working with Drives.

**Note:** If you connect the tape devices via HBA, Veeam Backup & Replication uses the block size configured for the HBA.

The block size is unified for:
- All drives in one library (if the drives support different block sizes)
- All standalone drives connected to one tape server.
- Mind the block size range when working with the following tapes:
- Tapes with Veeam backups written by another tape library,
- Tapes with Veeam backups written on another tape server,
- Tapes written with other data transfer configuration settings,
- Tapes written on a 3rd party device.

The tapes must be written with block size that match the value, currently used for the tape device you are using for restore.

## Sizing
For the highest throughput, enabling parallel processing for the Backup to Tape is recommended. You need to size the servers and storage connection accordingly. It can be helpful to create multiple partitions with 2-4 tape drives and add these partitions to different tape servers. Adding these libraries to the media pool and enabling parallel processing will distribute the load across multiple drives and tape servers.

Install Windows 2012 R2 or above on the tape server for best performance. Use the latest Veeam version and patch level as they often contain tape throughput optimizations.

Perform a POC to test throughput of tape and disk. If you have no opportunity to test speed, assume that the lowest speed for backup to tape jobs with LTO5/6 is 50MB/s as a conservative estimate. We highly recommend to do a POC to evaluate real throughput to avoid additional hardware costs.
