---
title: Enterprise Manager Restore operations
parent: Enterprise Manager
grand_parent: Veeam Components
nav_order: 10
---

# Using Enterprise Manager for Restore Operations

## Microsoft Exchange Mailbox Items Restore
The process of restoring an Exchange mailbox is described in the [Backup and Restore of Microsoft Exchange Items](https://helpcenter.veeam.com/docs/backup/em/em_exchange_items_restore.html) section of the Veeam Backup Enterprise Manager User Guide.

To create an application-aware image backup of Microsoft Exchange database VM ensure you back up at least one server holding the Client Access Server (CAS) role (This can be Exchange Server with the Mailbox Database role or a dedicated server. Contact the Exchange administrator if necessary). A server holding the CAS role is used to discover the mailbox location for the corresponding user. You should supply credentials for authentication with the CAS server on the **Configuration** > **Settings** page as described [here](https://helpcenter.veeam.com/docs/backup/em/em_providing_access_rights_exch.html).

## Self-Service File Restore
In addition to 1-Click File-Level Restore Backup & Replication allows VM administrators to restore files or folders from a VM guest OS using a browser from within the VM guest OS, without creating specific users or assigning them specific roles at the Veeam Enterprise Manager level. To do this an administrator of the VM can access the self-service web portal using the default URL: "https://ENTERPRISE_MANAGER:9443/selfrestore".

**Tip:** This feature is available only for the Windows-based VMs and requires Veeam Backup & Replication Enterprise *Plus* license. The VM needs to be in the same domain with the Enterprise Manager or in a trusted one (for SID resolution)

The process goes as follows:

1.  During the backup of a VM with guest indexing enabled, Veeam detects users who have local administrator access rights to that machine and stores this information in the Enterprise Manager database.
2.  User enters the self-service web portal URL in the web browser and enters the account name and password to access the necessary VM guest OS.
3.  After logging in the user is presented with the most recent restore point for that VM (the one this user authenticated to) on the **Files** tab of the web portal.

**Note:** This feature also works for backups from Veeam Agents for Windows stored on a Veeam Backup & Replication repository.

For more information on using this feature refer to the [Self-Restore of VM Guest Files](https://helpcenter.veeam.com/docs/backup/em/em_self_restore.html)
section of the Veeam Backup Enterprise Manager User Guide.