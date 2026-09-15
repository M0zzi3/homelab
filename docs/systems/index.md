# System catalogue

This catalogue documents the major VMs, LXCs and infrastructure systems by role. It is a public service map, not the complete operational inventory.

| System | Type | Main role | Documentation |
| --- | --- | --- | --- |
| Technitium DNS | Infrastructure guest | Internal DNS and custom local zone | [Technitium DNS](technitium-dns.md) |
| AI server | GPU-backed VM | Local model serving and media ML | [AI server](ai-server.md) |
| Docker deploy | Application VM | Shared Docker platform and delivery edge | [Docker deploy](docker-deploy.md) |
| Proxmox Backup Server | Backup system | Local recovery and off-site replication | [Proxmox Backup Server](proxmox-backup-server.md) |
| Gitea | Source-control LXC | Git repositories, reviews and public mirrors | [Gitea](gitea.md) |
| Hermes | Agent LXC | JARVIS runtime, dashboard and gateway | [Hermes](hermes.md) |
| ubuvault-alpha | Storage system | NAS and persistent media storage | [ubuvault-alpha](ubuvault-alpha.md) |
| Home Assistant | Automation guest | Smart-home control and integrations | [Home Assistant](home-assistant.md) |

Cross-system platforms:

- [Docker platform](../platforms/docker-platform.md)
- [JARVIS platform](../platforms/jarvis-platform.md)
- [Immich](../services/immich.md)

## Page standard

Each system page records:

- role and reason for existence;
- placement and dependencies;
- data ownership;
- operational responsibilities;
- security boundary;
- evidence still required;
- skills demonstrated.

A reusable outline is available in [the service-page template](../templates/service-page.md).
