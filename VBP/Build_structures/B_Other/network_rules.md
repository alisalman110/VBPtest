---
title: Network Rules
parent: other
grand_parent: 3-Build
nav_order: 60
---

# Network Rules

## Network Transport Encryption

Network encryption in Veeam Backup & Replication is controlled via the global Network Traffic options.

Whenever two backup infrastructure components need to communicate with each other over the IP
network, a dynamic key is generated by the backup server and communicated to each node over a
secure channel. The two components then establish an encrypted connection between each other using
this key, and all communications between these two components for that session are then encrypted
with this key. The key has a one-time use and it's discarded once the session is completed.

### When to use it?

Network transport encryption should be used if the network between two backup infrastructure
components is untrusted or if the user desires to protect Veeam traffic across the network from
potential network sniffing or "man in the middle" attacks.

By default, Veeam Backup & Replication automatically encrypts communication between two nodes if
either one or both has an interface configured (if used or not) that is not within the RFC1918
private address space (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.0.0/16). Veeam also
automatically uses network-level encryption for any connection with Veeam Cloud Connect service
providers. However, Cloud Connect establishes a TLS 1.2 encrypted tunnel to the service provider
in a different way.

### Best Practices

- Enable encryption if network-level attacks are a security concern.
- Network-level encryption can use significant CPU resources, especially on the encrypting side
  (source) of the connection. Make sure that component nodes have enough resources. Modern CPU's can
  offload encryption and reduce the amount of CPU resources required. For Intel CPU's specifically,
  you may check your CPU model on [Intel ARK](https://ark.intel.com) and look for the
  [AES-NI](https://en.wikipedia.org/wiki/AES_instruction_set) capability.
- Use network-level encryption only where required. If backup infrastructure components are running
  on a network that is using non-RFC1918 IP addresses but is still private and secure from attacks,
  consider using the following registry key to disable automatic network-layer encryption.
  - Path: `HKEY_LOCAL_MACHINE\SOFTWARE\Veeam\Veeam Backup and Replication`
  - Key: `DisablePublicIPTrafficEncryption`
  - Type: `REG_DWORD`
  - Value: `1` (_default: `0`_)

---

## References
- [Veeam Help Center - Network Traffic Management](https://helpcenter.veeam.com/docs/backup/vsphere/managing_network_traffic.html)
