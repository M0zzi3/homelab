# Proxmox Backup Server

## Role

The local Proxmox Backup Server stores deduplicated backups of protected VMs and LXCs and provides the first recovery path for the Belgium site.

## Backup topology

```mermaid
flowchart LR
    cluster[Proxmox cluster] --> local[Local PBS]
    local <-->|VPN replication path| poland[Second PBS in Poland]
```

The second PBS provides geographic separation. The exact sync initiator remains to be verified before the portfolio calls the relationship push or pull.

## Data ownership

PBS owns backup chunks, indexes and datastore metadata. Git-managed deployment files do not replace this data, and encryption-key handling must remain independent from the datastore itself.

## Operations

- monitor job completion and datastore growth;
- prune according to the intended retention policy;
- run verification and garbage collection;
- test representative restores;
- monitor the VPN and off-site sync relationship;
- document how recovery proceeds if the primary location is unavailable.

## Security boundary

The management interface, datastore paths, remote credentials, fingerprints, encryption keys and exact schedules remain private.

## Evidence to add

- sanitised retention policy;
- verification-job result;
- one measured VM or LXC restore;
- remote sync direction and failure behaviour;
- proof that encryption material is recoverable independently.

## Skills demonstrated

Proxmox Backup Server, deduplicated backup operations, retention, verification, restore testing, VPN-connected replication and disaster-recovery planning.

See [Backup and replication](../infrastructure/backup-and-replication.md) for the site-level design.
