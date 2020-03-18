---
title: Job Encryption
parent: Encryption
grand_parent: Supplemental
nav_order: 20
---
# Backup and Backup Copy Job Encryption

## What does it do?
Backup and backup copy job encryption is designed to protect data at rest. These settings protect data if an unauthorized user gets access to backup files, e.g. outside of the backup infrastructure. Authorized users of the Veeam console do not need to know the password to restore data from encrypted backups. Encryption does not prevent authorized Veeam users from being able to access data stored in backups.
An example is the use of rotated drives for an offsite repository. Because these drives are rotated offsite, they are at a higher risk of falling into the hands of unauthorized users. Without encryption enabled, these unauthorized users could install their own copy of Veeam Backup & Replication and gain access to the stored backups easily.
On the other hand, if the backup files are encrypted, unauthorized users cannot access any data in the backups or even learn any critical information about the backup infrastructure as even backup metadata is encrypted. Without the key used for encryption or access to the original Veeam Backup & Replication console itself, the backup files remain secure.

## How does it work?
To enable encryption functionality, backup encryption keys have to be generated. Those keys use mathematical symmetric cryptography and are not used to encrypt the data itself to avoid impacting backup performance. Instead, for each backup session a unique session symmetric encryption key is generated automatically and then stored in the backup file encrypted with the backup encryption key. Then each data block (compressed or not, depending on the job configuration) is encrypted using the session key previously generated for the current job session and stored in the backup file. In case Password Loss Protection functionality is enabled, an additional copy of session keys is stored in the backup file encrypted with the Enterprise Manager's encryption key.
This approach provides a method for encrypting backups without compromising backup performance.

## When to use it?
Backup and backup copy job encryption should be used if backups are transported offsite, or if unauthorized users may easily gain access to backup files in another way than by using the Veeam console. Common scenarios are:
- Offsite backups to a repository using rotated drives
- Offsite backups using unencrypted tapes
- Offsite backups to a Veeam Cloud Connect provider
- Regulatory or policy based requirements to store backups in encrypted form
Active full backup is required for enabling encryption to take effect if it was disabled for the job previously.

## Best Practices
- Enable encryption if you plan to store backups in locations outside of your security domain.
- While CPU usage for encryption is minimal for most modern processors, some amount of resources will still be consumed. If Veeam backup proxies are already highly loaded, take it into account prior to enabling job-level encryption.
- Use strong passwords for job encryption and develop a policy for changing them regularly. Veeam Backup & Replication helps with this as it tracks passwords' age.
- Store passwords in a secure location.
- Obtain Enterprise or a higher level license for Veeam Backup & Replication, configure Veeam Backup Enterprise Manager and connect backup servers to it to enable [Password Loss Protection][Decrypting Data Without Password].
- Export a copy of the active keyset from Enterprise Manager (see [User Guide][Enterprise Manager Encryption Keys] for more information).
- Back up the Veeam Backup Enterprise Manager configuration database and create an image-level backup of the Veeam Backup Enterprise Manager server. If these backups are also encrypted, make sure that passwords are not lost as there will be no Password Loss Protection for these backups.

----

## References
- [Decrypting Data Without Password]
- [Enterprise Manager Encryption Keys]

<!-- referenced links -->
[Decrypting Data Without Password]: https://helpcenter.veeam.com/docs/backup/vsphere/decrypt_without_pass.html
[Enterprise Manager Encryption Keys]: https://helpcenter.veeam.com/docs/backup/em/em_manage_keys.html
