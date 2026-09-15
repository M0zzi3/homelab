# Backup and off-site replication

## Objective

The backup design keeps recoverable Proxmox data locally while maintaining a second copy at another physical site in Poland. The two sites communicate through a private VPN path.

```mermaid
flowchart LR
    pve[Proxmox cluster in Belgium] -->|guest backups| local[Local Proxmox Backup Server]
    local <-->|encrypted VPN path| remote[Second Proxmox Backup Server in Poland]
    remote --> offsite[Off-site datastore]
```

## Local backup layer

The local Proxmox Backup Server is the first recovery target for protected VMs and LXCs. It is close enough for practical restores without depending on the remote site.

The documentation should record:

- protected guests;
- schedule and retention policy;
- datastore capacity;
- encryption ownership;
- pruning, verification and garbage-collection jobs;
- the latest tested restore for each important workload class.

## Off-site copy

A second Proxmox Backup Server in Poland stores an off-site copy over the VPN. This protects against failures that affect the primary location rather than only one VM or disk.

### Sync direction to verify

This page deliberately describes the path as replication rather than claiming a push or pull implementation. Proxmox Backup Server sync jobs are commonly initiated by the destination and pull from a configured remote. The final wording must match the running job configuration.

The following details still need verification:

- which PBS initiates the sync;
- which datastore is the source and destination;
- schedule and bandwidth controls;
- whether namespaces or filters limit the replicated backups;
- retention responsibility at the remote site;
- behaviour while the VPN or remote PBS is unavailable.

## Recovery model

| Failure | Preferred recovery path |
| --- | --- |
| Single guest failure | Restore from the local PBS |
| Local datastore failure | Restore from the off-site PBS after service is stabilised |
| Primary-site loss | Re-establish minimum compute and retrieve the remote backup set |
| Configuration-only regression | Roll back Git-managed configuration when data compatibility permits |
| Database migration failure | Use application-aware recovery rather than assuming a Compose rollback is sufficient |

## Evidence to add

- sanitised backup schedule and retention table;
- successful verification-job evidence;
- a measured test restore;
- a sanitised view of the remote relationship and sync job;
- recovery notes for the most important systems;
- handling of encryption keys independently from the backup datastore.

## Skills demonstrated

Proxmox Backup Server, retention planning, remote replication, VPN transport, restore testing, failure-domain design and disaster-recovery documentation.
