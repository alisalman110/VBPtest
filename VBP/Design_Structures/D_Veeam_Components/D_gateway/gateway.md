---
title: Veeam Gateway Server
parent: Veeam Components Designs
grand_parent: Design
nav_order: 10
---

# Veeam Gateway Server 

Introduction and see link at bottom of page

## Gateway Placement

Where the gateway should be placed.

## Gateway Server Requirements 

Server OS requirements.

## Sizing

A server hosting the proxy and gateway role should theoretically be sized to sustain both loads. For example 1 core & 2 GB RAM proxy plus 1 core & 4 GB RAM gateway for each parallel stream (a.k.a. VMDK or VHD). In this situation a conservative rule would be 2 core and 6 GB of RAM per parallel task on the machine hosting the proxy and gateway roles but a less conservative rule can be applied if the job compression is set to "dedupe friendly" or "none". The CPU will be spared, and the rule can be set to 1 core and 4 GB RAM per parallel task.

If the gateway is not coupled with the proxy its sizing is the same than for any simple repository.

<!-- This has been taken from B_backup_repositories > gateways.md -->

## References