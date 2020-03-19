---
title: Backup Job Automation
parent: Veeam Backup Jobs
grand_parent: Veeam Jobs
nav_order: 60
---
## Backup Job Automation

### Container based jobs

When creating jobs based on groups or constructs, ensure that the configured constructs do not overlap.

#### Tags
It is recommended to follow these guidelines:

-   Monitor the number of VMs automatically added to the job to avoid too many VMs being backed up within a single job
-   Only one tag can be used to include a VM in a job
-   Using tags, you can classify VMs by service levels, using different backup jobs for different service levels
-   Veeam ONE Business View (OBV) is a very convenient tool for managing vSphere Tags. OBV allows for creating classification rules and update corresponding
tags in vCenter. Classifications can be defined from CPU, RAM, VM naming convention, folder, resource pool, datastore etc. OBV can also import VM/host/datastore descriptions from a CSV file. This feature can be useful when refreshing VMware tags, for example, to update a CMDB.

<hr>

###  Container Based Jobs

Adding resource pools, folders, datastores, or vSphere Tags (vSphere 5.5 and higher) to backup jobs makes backup management easier. New machines that are
member of such constructs or containers are automatically included in the backup job, and machines removed from the container are immediately removed from
job processing.

Overlapping constructs may cause undesired results. For instance, when creating jobs based on datastores, VMs with disks residing on multiple datastores included in more than one backup job will cause the VM to be backed up in each job.

<hr>

### References

[Container Jobs](https://helpcenter.veeam.com/docs/backup/vsphere/backup_job_vms_vm.html?ver=100) - see reference to toolbar to switch between views

[VM Tags](https://helpcenter.veeam.com/docs/backup/vsphere/vm_tags.html?ver=100)
