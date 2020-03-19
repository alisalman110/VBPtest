---
title: Veeam Backup Enterprise Manager
parent: Enterprise Manager
grand_parent: Veeam Components
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
