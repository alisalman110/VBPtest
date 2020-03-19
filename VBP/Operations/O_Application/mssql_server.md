---
title: Microsoft SQL Server
parent: Applications
grand_parent: 4-Operate
nav_order: 40
---



# Microsoft SQL Server

Veeam Backup and Replication provides two options for backing up SQL Server:

- Image-level backup of the virtual machines running SQL Server
- Backup of Windows machine running SQL Server using Veeam Agent

Two options differ in capabilities and the list of supported configurations.

|                              | VM backup     | Agent backup |
| ------                       | ------        | ------       |
| Standalone SQL server        | Supported     | Supported    |
| Always On Availability Group | Supported     | Supported    |
| SQL Failover Cluster         | Not supported | Supported    |


*Note:* Please read the whitepaper on benefits of using [SQL Always-on Availability Groups for Virtual Environment](https://www.veeam.com/wp-sql-alwayson-availability-groups-virtual-environment.html)

Veeam Agent has additional limitations applied to specific configurations:

- Backup of CSV (Cluster Shared Volumes) is not supported. Cluster disks used as CSV are automatically excluded from backup.
- AlwaysOn Availability Groups based on multiple Failover Cluster Instances are not supported.
- AlwaysOn Clusterless Availability Groups are not supported.

## Preparation

The following section will provide the best practices to verify SQL Server for the smooth and fast backup.

Check the status of VSS writers and providers:

1.	Open a command prompt as Administrator.
2.	Type “vssadmin list providers” and press enter.
3.	Verify all the writers are in health state.
4.	Type “vssadmin list writers” and press enter
5.	Check and confirm writers are “Stable with no errors”.

As backup is I/O intensive operations check the SQL server performance status before start taking the backup. To verify the SQL Server Health:

1. Open Performance Monitor.
2. Add below objects verify the performance:
  1. Under the Memory object, add the counter: Available MB
  1. Under the Processor object, add the counter: %Processor Time (Instance: Total)
  1. Under the Physical Disk object, add the counters: Avg. Disk Sec/Read and Avg. Disk Sec/Write (All Instances)
  1. Under the Paging File object, add the counter: %Usage (Instance: Total)

Check Microsoft article ["Monitor Memory Usage"](https://docs.microsoft.com/en-us/sql/relational-databases/performance-monitor/monitor-memory-usage?view=sql-server-2017) to validate the health status of SQL Server.

## Job Configuration

## Virtual Machine Image Level Backup

### Standalone SQL Server

No additional configuration is required to backup the standalone SQL server, you can configure the backup with application aware processing to take the backup of SQL server with the databases.

Please check the Veeam User’s Guide sections to get the more information about [SQL backup configuration](https://helpcenter.veeam.com/docs/backup/explorers/vesql_bu_job_settings.html?ver=95)

### SQL Always-on Availability Group

When backing up AlwaysOn availability group make sure all cluster nodes are processed by the same backup job for transaction logs processing and restores to work properly. Consider increasing cluster timeouts in case failover occurs during the backup, similar to Exchange DAG as per KB1744.

You can also follow the steps outlined in [KB2110](https://www.veeam.com/kb2110) to exclude certain databases from application aware processing.

## Transactions Logs Backup:

Please be aware that transactions logs are processed periodically and stored in temporary folder inside of the VM before shipping to repository/shipping server. Default location of the temporary folder is %allusersprofile%\Veeam\Backup. The default location is in most cases the system partition to avoid the situation to run out of space in the system partition, it’s the best practices to change the temporary folder location to the windows disk volume where enough space is available for staging the transactions logs.

Temporary folder path can be changed using SqlTempLogPath (STRING) registry value:

- Path: HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication
- Key: SqlTempLogPath
- Type: REG_SZ
- Default value: undefined

For instructions on changing temporary folder path for agent-based backups refer to [KB2642 article](https://www.veeam.com/kb2642).

As best practices it's highly recommended to periodically shrink the SQL log file, Please follow [Microsoft article](https://docs.microsoft.com/en-us/sql/relational-databases/databases/shrink-a-file?view=sql-server-ver15) for more information.


## Restore

Restore is integrated part of SQL Server protection, Veeam provides following options to restore SQL Server:

1.	VM or Physical Server Restore.
      - Instant VM or Server Restore.
      - BMR Restore for Physical.
      - Full VM Restore.
2.	SQL Application Item Level Restore.

Veeam uses specially designed Veeam Explorer for SQL to perform application item level restore, to optimize the restore of SQL database install Veeam Management console on SQL server locally to perform the restores.

### SQL Failover Cluster Database Restore: (Applicable to agent based cluster backup only)

#### SQL Database Restore:

Please follow the below steps to restore SQL database:
1.	File Level Restore to local Veeam Server.
2.	Copy the mdf and ldf file to target SQL Server.
3.	Attach the Database to target SQL Server.

#### SQL Table Level Restore:

1.	Restore SQL Database to Staging SQL Server(Stand alone SQL Node)
2.	Open SQL Server Management Studio. Connect both staging and target SQL server(intend to do the restore)
3.	Right-click on the database name, then select "Tasks" > "Export data..." from the object explorer.
4.	The SQL Server Import/Export wizard opens; click on "Next".
5.	Provide authentication and select the staging server; click "Next"
6.	Specify the target SQL Server; click on "Next".
7.	Select the tables to restore.
8.	Complete the restore.
