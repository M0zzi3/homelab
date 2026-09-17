# Maksymilian's homelab

A multi-node homelab where I practise infrastructure engineering, networking, automation and self-hosted AI beyond isolated classroom exercises.

I use it to design services, deploy them, troubleshoot failures and improve how they are recovered. This repository is a sanitised portfolio view; the live inventory, credentials and deployment configuration remain private.

## Overview

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart LR
    %% Network Edge & Ingress
    subgraph NetworkEdge ["Network Edge"]
        Internet(("Public Internet")) --> OpenWrt["OpenWrt Firewall / Router"]
        WireGuard["WireGuard VPN"] --> OpenWrt
        OpenWrt --> Switch["TP-Link Managed Switch"]
    end

    %% Proxmox VE Cluster
    subgraph ProxmoxCluster ["Proxmox VE Cluster"]
        subgraph Node1 ["Proxmox Node 1"]
            PBS[("Proxmox Backup Server")]
            NAS[("ubuvault-alpha NAS")]
            HA["Home Assistant"]
        end

        subgraph Node2 ["Proxmox Node 2 (endurance)"]
            AIServer["AI Server (GPU Accelerated)"]
        end

        subgraph HAServices ["High Availability Services Pool"]
            DockerServer["Docker Server"]
            Technitium["Technitium DNS"]
            Gitea["Gitea Local Git"]
            Hermes["Hermes AI Agent"]
            Tailscale["Tailscale Exit Node"]
        end
    end

    %% Docker Platform Applications
    subgraph DockerApps ["Docker Applications"]
        Traefik["Traefik Proxy Server"]
        Dockhand["Dockhand Management"]
        n8n["n8n Automation Platform"]
        Immich["Immich Photo Library"]
        MCP["MCP Stack"]
        Honcho["Honcho AI Memory"]
    end

    %% Off-Site Disaster Recovery
    subgraph Offsite ["Off-Site Disaster Recovery"]
        PolandPBS[("Off-site PBS (Poland)")]
    end

    %% Ingress to Compute Nodes
    Switch --> Node1
    Switch --> Node2

    %% Nodes feed shared HA Services Pool
    Node1 --> HAServices
    Node2 --> HAServices

    %% Docker Container Stacks
    DockerServer --> Traefik
    DockerServer --> Dockhand
    DockerServer --> n8n
    DockerServer --> Immich
    DockerServer --> MCP
    DockerServer --> Honcho

    %% Off-site Disaster Recovery Link
    PBS ===|"Encrypted VPN Tunnel"| PolandPBS

    %% Styling & Color Coding (Dark Theme)
    style NetworkEdge fill:#0f172a,stroke:#64748b,stroke-width:2px,color:#e2e8f0;
    style ProxmoxCluster fill:#181024,stroke:#a855f7,stroke-width:2px,color:#e2e8f0;
    style Node1 fill:#260d36,stroke:#c084fc,stroke-width:1.5px,color:#e2e8f0;
    style Node2 fill:#2e1a05,stroke:#fb923c,stroke-width:1.5px,color:#e2e8f0;
    style HAServices fill:#0c1f2e,stroke:#38bdf8,stroke-width:1.5px,color:#e2e8f0;
    style DockerApps fill:#042116,stroke:#10b981,stroke-width:2px,color:#e2e8f0;
    style Offsite fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#e2e8f0;

    classDef edgeNode fill:#1e293b,stroke:#94a3b8,stroke-width:1.5px,color:#f8fafc;
    classDef storageNode fill:#3b0764,stroke:#c084fc,stroke-width:1.5px,color:#f8fafc;
    classDef computeNode fill:#451a03,stroke:#fb923c,stroke-width:1.5px,color:#f8fafc;
    classDef aiNode fill:#1e1b4b,stroke:#818cf8,stroke-width:1.5px,color:#f8fafc;
    classDef netNode fill:#0c4a6e,stroke:#38bdf8,stroke-width:1.5px,color:#f8fafc;
    classDef dockerNode fill:#064e3b,stroke:#34d399,stroke-width:1.5px,color:#f8fafc;
    classDef offsiteNode fill:#311b92,stroke:#818cf8,stroke-width:1.5px,color:#f8fafc;

    class Internet,WireGuard,OpenWrt,Switch edgeNode;
    class PBS,NAS storageNode;
    class HA computeNode;
    class AIServer,Hermes,MCP,Honcho,n8n aiNode;
    class Technitium,Tailscale netNode;
    class DockerServer,Traefik,Dockhand,Immich,Gitea dockerNode;
    class PolandPBS offsiteNode;
```

The environment is built around:

- a multi-node Proxmox cluster with a separate cluster network;
- OpenWrt, managed switching, VLAN segmentation, internal DNS and WireGuard;
- a central Docker host using Gitea, Dockhand and Traefik;
- local and off-site Proxmox backups;
- a NAS used by stateful applications such as Immich;
- a distributed JARVIS stack using Hermes, n8n, MCP, Honcho and local AI.

## What this demonstrates

| Area | Evidence |
| --- | --- |
| Virtualisation and networking | [Architecture](docs/architecture.md) and [Proxmox case study](docs/case-studies/proxmox-platform.md) |
| Systems and data ownership | [System catalogue](docs/systems.md) |
| Deployment, backup and recovery | [Operations](docs/operations.md) and [GitOps case study](docs/case-studies/gitops-deployment.md) |
| Local AI and agent automation | [Self-hosted AI case study](docs/case-studies/self-hosted-ai.md) |

## Main systems

| System | Purpose |
| --- | --- |
| Technitium DNS | Internal DNS and custom local service names |
| Docker deploy | Container platform, CI/CD and reverse proxy |
| AI server | Local model serving and Immich machine learning |
| Gitea | Source control, reviews and GitHub mirrors |
| Hermes | JARVIS runtime and tool orchestration |
| Home Assistant | Smart-home control and local integrations |
| ubuvault-alpha | NAS and persistent media storage |
| Proxmox Backup Server | Local recovery and off-site replication to Poland |

The [system catalogue](docs/systems.md) explains where each system fits without exposing operational access details.

## Featured case studies

### [Proxmox platform](docs/case-studies/proxmox-platform.md)

How I separated compute, cluster communication, services and recovery concerns in a constrained home environment.

### [Git-managed Docker deployments](docs/case-studies/gitops-deployment.md)

How configuration moves from a Gitea branch through review, Dockhand deployment and post-deployment verification.

### [Self-hosted AI](docs/case-studies/self-hosted-ai.md)

How I balance local language models, memory services and Immich ML on limited GPU, memory and storage capacity.

## Working principles

- inspect the live system before changing it;
- keep non-secret configuration in Git;
- define a rollback path before consequential changes;
- verify useful behaviour, not only process status;
- document failures and trade-offs honestly.

## Security

This repository does not contain credentials, private keys, live addresses, complete firewall or VPN configuration, personal data, private automation payloads or exact backup destinations. See [SECURITY.md](SECURITY.md).

## Current work

The foundational architecture and subsystem diagrams have been integrated. The next additions are verified Proxmox resource allocations, selected sanitised configuration examples and measured recovery evidence.

## Author

[Maksymilian Piast](https://github.com/M0zzi3)<br>
Final-year Electronics-ICT student, specialising in System, Services and Security.
