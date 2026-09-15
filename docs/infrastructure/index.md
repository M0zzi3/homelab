# Infrastructure

The infrastructure documentation covers the shared layers that every hosted system depends on.

| Area | Scope | Documentation |
| --- | --- | --- |
| Proxmox cluster | Compute nodes, quorum, guest placement, memory planning and HA | [Proxmox cluster](proxmox-cluster.md) |
| Network | OpenWrt, managed switching, VLANs, DNS and remote access | [Network architecture](network.md) |
| Backup | Local Proxmox Backup Server and the off-site copy in Poland | [Backup and replication](backup-and-replication.md) |

Individual VMs and LXCs are documented in the [system catalogue](../systems/index.md). Platforms spanning several systems are documented separately under [Docker platform](../platforms/docker-platform.md) and [JARVIS platform](../platforms/jarvis-platform.md).
