---
title: Encryption Overview
parent: Encryption
grand_parent: Supplemental
nav_order: 10
---

# Encryption

## Overview
The encryption technology in Veeam Backup & Replication allows to protect data both while it is in transfer between backup components and at rest, when stored at its final destination (backup repository, tape, cloud repository or object storage). Customers can use one of the encryption methods or a combination of both to protect against unauthorized access to important data through all the steps in the data protection process.

Veeam Backup Enterprise Manager additionally provides Password Loss Protection option that allows authorized Veeam users to recover data from the backup even if the encryption password is lost. If the password gets lost, the backup server will provide a challenge key for Enterprise Manager. Using asymmetric encryption with a public/private key pair, Enterprise Manager generates a response which the backup server can use for unlocking the backup file without having the password available. For more details on this feature, refer to the [corresponding section of the User Guide][Decrypting Data Without Password].

The encryption algorithms used are industry standard in all cases, leveraging AES-256 and public key encryption methods. The [Data Encryption] section of the User Guide provides detailed information on the encryption algorithms and standards used by the product.

----

## References
- [Decrypting Data Without Password]
- [Data Encryption]

<!-- referenced links -->
[Decrypting Data Without Password]: https://helpcenter.veeam.com/docs/backup/vsphere/decrypt_without_pass.html
[Data Encryption]: https://helpcenter.veeam.com/docs/backup/vsphere/data_encryption.html
