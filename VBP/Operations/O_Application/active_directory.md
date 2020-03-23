---
title: Active Directory
parent: Applications
grand_parent: Operate
nav_order: 20
---



# Active Directory

Veeam supports Application Aware backup of Active Directory for Virtual Machine and Physical Servers. 

## Preparation

For Microsoft Active Directory, check the tombstone lifetime settings, as described in Veeam Explorers User Guide at Veeam Help Center (https://helpcenter.veeam.com/docs/backup/explorers/vead_recommendations.html?ver=95).

When possible, it's recommended to backup the Domain Controller with most FSMO

You can run netdom query fsmo to check with Domain Controller have which FSMO roles.

Learn more about FSMO Roles (https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/planning-operations-master-role-placement)

## Job configuration

For backup and restore of domain controllers to work properly application aware image processing option has to be enabled in the job properties. For more details refer to the [corresponding section](https://helpcenter.veeam.com/docs/backup/vsphere/backup_job_vss_vm.html?ver=95) of the User Guide.

## Restore and failover

After the restore, you might need to use ntdsutil seize command to transfer the FSMO roles, it's recommended to deploy mutiple domain controllers for high availability and redundancy. 

More information about ntdsutil seize(https://support.microsoft.com/en-us/help/255504/using-ntdsutil-exe-to-transfer-or-seize-fsmo-roles-to-a-domain-control)

## Recovery verification

There are two Domain Controller roles available in application group configuration - for authoritative and non-authoritative restore. When testing recovery of one domain controller only choosing role with authoritative restore will speed up verification process.
