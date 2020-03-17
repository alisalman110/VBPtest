---
title: Interaction with Guest OS
parent: Hyper-V
grand_parent: Supplemental
nav_order: 30
---

# Interaction with Guest OS

## PowerShell Direct
Introduced by Microsoft in Hyper-V 2016, PowerShell Direct is a new method allowing to interact with the guest even if no direct network connection is available between the Veeam guest interaction proxy and the guest itself.

PowerShell Direct requires the following prerequisites:
- PowerShell 2.0 or later
- Host must be Windows Server 2016 or later
- Guest must be Windows Server 2016/Windows 10 or later

PowerShell Direct can be easily tested on the host, using the following command.

```PowerShell
Enter-PSSession -VMName <VMName>
```

---

## References
- [Best Practice - Guest Interaction](anatomy/guestinteraction.md) (General guest interaction info)
- [User Guide - Guest Processing](https://helpcenter.veeam.com/docs/backup/hyperv/guest_processing.html)
