# Maksymilian's homelab

A multi-node homelab used to practise infrastructure engineering, network design, reliable deployments, automation and self-hosted AI.

This repository is a public, sanitised view of the environment. It focuses on the engineering decisions, operational lessons and selected examples that can be shared safely. Exact inventory, credentials and production configuration remain private.

## Why I built it

Coursework gave me isolated labs. I wanted an environment where services depend on each other, changes have consequences, and recovery matters as much as deployment.

The lab lets me work with:

- Linux systems, virtual machines and containers;
- segmented networking, internal DNS and remote access;
- Git-managed Docker deployments;
- backups, health checks and rollback procedures;
- workflow automation across APIs and services;
- local language models and GPU-constrained AI workloads.

## Architecture at a glance

```mermaid
flowchart TB
    users[Trusted clients and remote access] --> edge[Router and firewall]

    edge --> core[Core services network]
    edge --> iot[IoT network]
    edge --> guest[Guest network]
    edge --> cluster[Cluster communication network]

    core --> dns[Authoritative internal DNS]
    core --> pve[Proxmox cluster]

    subgraph virtualisation[Virtualisation and recovery]
        pve --> guests[VMs and LXCs]
        pve --> backup[Proxmox Backup Server]
    end

    guests --> git[Gitea]
    guests --> docker[Docker deployment host]
    guests --> ai[AI compute host]
    guests --> automation[Hermes and Home Assistant]
    guests --> storage[NAS storage]

    subgraph platform[Container platform]
        docker --> proxy[Traefik]
        docker --> deploy[Dockhand]
        docker --> workflows[n8n]
        docker --> memory[Honcho]
        docker --> gateway[MCP gateway]
        docker --> photos[Immich core]
    end

    subgraph local_ai[Local AI workloads]
        ai --> models[LM Studio model serving]
        ai --> media_ml[Immich machine learning]
    end

    workflows --> gateway
    automation --> workflows
    memory --> models
    photos --> media_ml
    photos --> storage
    backup --> offsite[Second PBS over VPN]
```

The diagram shows roles and trust boundaries rather than operational addresses or hostnames. See [Architecture](docs/architecture.md) for the expanded view.

## Explore the system

- **Infrastructure:** [Proxmox cluster](docs/infrastructure/proxmox-cluster.md), [network architecture](docs/infrastructure/network.md), and [backup replication](docs/infrastructure/backup-and-replication.md)
- **VMs and LXCs:** [system catalogue](docs/systems/index.md) with a page for each major host or guest
- **Platforms:** [Docker delivery](docs/platforms/docker-platform.md) and the distributed [JARVIS platform](docs/platforms/jarvis-platform.md)
- **Applications:** [Immich](docs/services/immich.md), including its Docker, NAS and GPU dependencies

## Technical highlights

| Area | What I implemented | Evidence |
| --- | --- | --- |
| Virtualisation | Multi-node Proxmox environment with VMs, LXCs and a separate cluster network | [Proxmox cluster](docs/infrastructure/proxmox-cluster.md) |
| Networking | OpenWrt, managed switching, VLAN boundaries, internal DNS and WireGuard | [Network architecture](docs/infrastructure/network.md) |
| Delivery | Gitea-based configuration workflow, reviewed changes, Dockhand deployment and Traefik routing | [Docker platform](docs/platforms/docker-platform.md) |
| Recovery | Local PBS backups and a second PBS in Poland connected over VPN | [Backup and replication](docs/infrastructure/backup-and-replication.md) |
| Automation | Hermes, n8n, Honcho and MCP services working across several hosts | [JARVIS platform](docs/platforms/jarvis-platform.md) |
| AI operations | Local model serving, memory services and GPU capacity planning alongside Immich ML | [AI server](docs/systems/ai-server.md) |

## Case studies

### [Proxmox platform and segmented networking](docs/case-studies/proxmox-platform.md)

How I separated infrastructure roles, cluster traffic, services and recovery concerns across a constrained home environment.

### [Git-managed Docker deployments](docs/case-studies/gitops-deployment.md)

How Gitea, pull requests, Dockhand and runtime health checks provide a safer path from configuration change to running service.

### [Self-hosted AI under resource constraints](docs/case-studies/self-hosted-ai.md)

How I separated AI compute, selected lightweight local models and balanced persistent memory workloads against GPU capacity needed by other services.

## How I approach infrastructure work

1. Inspect the live system before changing it.
2. Record the current state and identify a rollback path.
3. Keep non-secret configuration in Git.
4. Separate runtime secrets from version-controlled files.
5. Change one layer at a time when possible.
6. Verify the result from the service and client side.
7. Document failures and the reasoning behind the final design.

The lab is intentionally imperfect. Its purpose is to make constraints, failures and trade-offs visible enough to learn from them.

## Skills demonstrated

- Proxmox VE, KVM and LXC
- Linux system administration
- Docker and Docker Compose
- Git, Gitea and pull-request workflows
- network segmentation, VLANs, DNS and VPN access
- Traefik and internal service routing
- backup and recovery planning
- n8n workflow automation and API integration
- local LLM serving and AI workload planning
- technical documentation and incident analysis

## Repository map

```text
docs/
├── architecture.md
├── engineering-decisions.md
├── security-and-privacy.md
├── infrastructure/
│   ├── proxmox-cluster.md
│   ├── network.md
│   └── backup-and-replication.md
├── systems/
│   ├── index.md
│   └── one page per major VM, LXC or host
├── platforms/
│   ├── docker-platform.md
│   └── jarvis-platform.md
├── services/
│   └── immich.md
├── case-studies/
│   ├── proxmox-platform.md
│   ├── gitops-deployment.md
│   └── self-hosted-ai.md
└── templates/
    └── service-page.md

examples/
└── README.md
```

The `examples` directory will contain selected, sanitised configuration fragments. The repository will not mirror the private deployment repositories.

## Security and privacy

This repository deliberately excludes:

- credentials, tokens, private keys and certificate material;
- real public addresses, MAC addresses and device serial numbers;
- complete firewall, VPN or remote-access configurations;
- unredacted environment files and application exports;
- backup destinations and other details that would meaningfully reduce security.

The full policy and publication checklist are in [Security and privacy](docs/security-and-privacy.md).

## Current status

The information architecture and first system pages are in place. The next additions are verified resource allocations, sanitised configuration examples, screenshots and measured recovery evidence.

## Author

[Maksymilian Piast](https://github.com/M0zzi3)<br>
Final-year Electronics-ICT student, specialising in System, Services and Security.
