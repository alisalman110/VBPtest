---
<<<<<<< HEAD
title: Veeam Enterprise Manager
parent: Veeam Components Designs
grand_parent: Design
=======
title: Veeam Enterprise Manager Overview
parent: Veeam Enterprise Manager
grand_parent: Veeam Components Designs
>>>>>>> Stashed changes
nav_order: 10
---

# Veeam Backup Enterprise Manager

## Deployment use cases

It is recommended to deploy Enterprise Manager if:
- you are using encryption for backup or backup copy jobs. If you have enabled [password loss protection](https://helpcenter.veeam.com/docs/backup/em/em_manage_keys.html). Enterprise Managers administrators will have the possibility to unlock backup files using a challenge/response mechanism effectively acting as a Public Key Infrastructure (PKI).
- your organization has a Remote Office/Branch Office (ROBO) deployment. Sites administrators will have acces to the granular restore via web UI rather than providing access to Backup & Replication console.
- delegation capabilities must be used to elevate the 1st line support to perform in-place restores without administrative access.
- you need to centrally manage licenses for multiple Veeam Backup & Replication servers.
- you are planing extensive file restores from large file servers. You can then enhance the user experience through the file indexing engine.
- you plan to use Veeam RESTful API to automate your data protection workloads.

If the environment includes a single instance of Backup & Replication you may not need to deploy Enterprise Manager, especially if you want to avoid additional SQL Server database activity and server resource consumption (which can be especially important if using SQL Server Express Edition).

## Enterprise Manager server Placement

<<<<<<< HEAD
If possible, it is advised to install the Enterprise Manager server on the recovery site so it is directly available in case of Disaster revovery.

## Sizing Enterprise Manager server compute

The Veeam Enteerprise Manager server will very rarely necessitate more than 8 cores and 16 GB of RAM.
=======
If possible, it is advised to install the Enterprise Manager server on the recovery site so it is directly available in case of Disaster revovery.

## Sizing Enterprise Manager server compute

The Veeam Enteerprise Manager server will very rarely necessitate more than 8 cores and 16 GB of RAM.
>>>>>>> Stashed changes

## Sizing Enterprise Manager Catalog

The Veeam Catalog Service is responsible for maintaining index data. When running on the backup server this catalog service will maintain index data for all jobs that run on that specific server as long as the backup data remains on disk. When running on the Enterprise Manager server the service will move index data from all managed backup servers into the Enterprise Manager local catalog deleting the files from the originating backup server catalog. So it should be sized appropriately to hold all data from the remote Veeam servers.

- When using a Standard license, Enterprise Manager will only keep index data for restore points still in repositories.
- For Enterprise and Enterprise Plus licenses, you can configure Enterprise Manager to keep indexes even longer, with the default being 3 months. This can significantly increase the amount of space required for the catalog.

Estimated used space of the final index file after compression is approximately 2 MB per 1,000,000 files for a single VM restore point on the Enterprise Manager server. The indexes are also stored in the backup files and temporary folders on the backup server.

Below is an example that summarizes the information above. The example is given per indexed VM containing 10,000,000 files.

2 MB * 10 million files * 60 restore points per month * 3 months index retention = 3.5 GB

## References
[Veeam backup Enterprise Manager Guide](https://helpcenter.veeam.com/docs/backup/em/introduction.html?ver=100)

[Veeam Backup Catalogue](https://helpcenter.veeam.com/docs/backup/em/veeam_backup_catalog.html?ver=100)


<!--
# Veeam Enterprise Manager

Introduction and see link at bottom of page

<<<<<<< HEAD
## Enterprise Manager Placement
=======
## Enterprise Manager Placement
>>>>>>> Stashed changes

Information on the placement of Enterprise Manager.

##  Enterprise Manager OS requirements

OS requirements

## Enterprise Manager Sizing

<<<<<<< HEAD
Enterprise Manager base requirements, plus additional overheads for Self-service, API and multi-VBR.
=======
Enterprise Manager base requirements, plus additional overheads for Self-service, API and multi-VBR.
>>>>>>> Stashed changes

Need to add info on the EM DB requirements.

## References
<<<<<<< HEAD
-->
=======
-->
>>>>>>> Stashed changes
