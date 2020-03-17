---
title: Instant VM Recovery
parent: Hyper-V
grand_parent: Supplemental
nav_order: 60
---

# Instant VM Recovery

## Storage Requirement
When performing [Instant VM recovery](https://helpcenter.veeam.com/docs/backup/hyperv/instant_recovery.html), Veeam will immediately pre-allocate the necessary amount of storage on the target infrastructure, even though the guest image used is residing on the backup repository.

**Note :** this pre-allocation is performed only for Instant VM Recovery Usage. Sure Backup processing will use a thin provisioning mechanism instead, preserving resources on the infrastructure.

**Note :**  Instant VM Recovery, will send Data over the production (DNS aware) network. During "recover to production" the prefered network are not used for data traffic. It's recommended to use a fast network connection or the Hyper-V parent partition for fast IVMR and recovery to production.

---

## References
- [Veeam Helpcenter - Hyper-V Instant VM Recovery](https://helpcenter.veeam.com/docs/backup/hyperv/instant_recovery.html)
