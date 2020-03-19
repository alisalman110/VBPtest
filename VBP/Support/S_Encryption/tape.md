---
title: Tape Job Encryption
parent: Encryption
grand_parent: Supplemental
nav_order: 50
---
# Tape Job Encryption

## What does it do?
Similar to [backup job encryption](./backup-and-copy-job-encryption.md), tape job encryption is designed to protect data at rest. It is to protect data in the case of an unauthorized user gaining access to tape media outside of the backup infrastructure. Authorized users do not need to know the password to restore data from encrypted tape backups. Encryption does not prevent authorized Veeam users from being able to access data stored in tape backups.
The typical use case is to protect data on tapes when media is shipped to an offsite or 3rd party location. Without encryption enabled, a lost tape could easily be accessed, and data stored on tapes could be compromised.

## How does it work?
Similar to encryption for backups on disk, a session encryption key is used to encrypt data blocks as they are written to tape. Tape encryption can leverage either hardware tape encryption (if present and enabled) or software-based encryption. If the tape drive supports hardware encryption, the session key is sent to the tape device via SCSI commands and the drive itself performs the encryption prior to writing data to tape. This allows encryption to occur with no impact on the CPU of the tape server. If the tape hardware does not support encryption, Veeam falls back to using software-based AES-256 data encryption prior to sending data to the tape device.

## When to use it?
Tape job encryption should be used any time you want to protect the data stored on tape from unauthorized access by a 3rd party. Tapes are commonly transported offsite and thus have a higher chance of being lost or stolen. Encrypting tapes can provide an additional layer of protection for such cases.
If tape jobs are writing already encrypted data to tape (for example, Veeam data from backup jobs that already have encryption enabled), you may find it acceptable to not use tape-level encryption. However, be aware that a user who gets access to the tape will be able to restore the backup files. Although this user will not be able to access the contents of those files, some valuable information (e.g. job names used for backup files) may leak.

## Best Practices
- Enable encryption if you plan to store tapes in locations outside of your security domain.
- Consider the risks/benefits of enabling tape job encryption even if the source data is already encrypted and evaluate appropriately the acceptable level of risk.
- Use strong passwords for tape job encryption and develop a policy for changing them regularly (you can use Veeam Backup & Replication password age tracking capability).
- Store passwords in a secure location.
- Obtain Enterprise or a higher level license for Veeam Backup & Replication, configure Veeam Backup Enterprise Manager and connect backup servers to it to enable [Password Loss Protection][Decrypting Data Without Password].
- Back up the Veeam Backup Enterprise Manager configuration database and create an image-level backup of the Veeam Backup Enterprise Manager server. If these backups are also encrypted, make sure that passwords are not lost as there will be no Password Loss Protection for these backups.

----

## References
- [Decrypting Data Without Password]

<!-- referenced links -->
[Decrypting Data Without Password]: https://helpcenter.veeam.com/docs/backup/vsphere/decrypt_without_pass.html
