# Architecture

This page describes the homelab at a portfolio level. It explains the main boundaries and dependencies without reproducing the live address plan or private configuration.

## Architecture layers

### Network

OpenWrt provides routing, firewalling and DHCP. A managed switch carries the required VLANs. Technitium provides internal DNS, and WireGuard provides authenticated remote access.

The public model shows four network purposes:

- core infrastructure and trusted clients;
- IoT devices;
- guest access;
- Proxmox cluster communication.

### Compute and storage

Proxmox hosts the main VMs and LXCs. Workloads are separated by responsibility: infrastructure, container applications, AI compute, home automation, JARVIS, NAS storage and backup.

A local Proxmox Backup Server provides the first restore path. A second PBS in Poland stores an off-site copy over the VPN. The final documentation will name the sync direction only after the running job has been checked.

### Applications and automation

The Docker host runs Traefik, Dockhand, n8n, the MCP stack, Honcho and Immich core services. Immich stores its media on the NAS and sends machine-learning work to the AI server.

Hermes is the visible JARVIS runtime. n8n owns durable automation, MCP exposes authorised integrations, Honcho handles conversational memory, and the AI server provides local inference.

## Diagram 1: homelab overview

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart LR
    %% Edge & Network
    subgraph External ["External & Remote Access"]
        Internet(("Internet"))
        WireGuard["WireGuard VPN"]
        PolandPBS[("Off-site PBS (Poland)")]
    end

    subgraph Network ["Network Edge"]
        OpenWrt["OpenWrt Firewall / Router"]
        Switch["TP-Link Managed Switch"]
    end

    %% Proxmox Cluster Subgraph
    subgraph Proxmox ["Proxmox VE Cluster"]
        subgraph Node1 ["Proxmox Node 1"]
            LocalPBS[("Proxmox Backup Server")]
            NAS[("NAS Server")]
            HomeAssistant["Home Assistant"]
        end

        subgraph Node2 ["Proxmox Node 2 (endurance)"]
            AIServer["AI Server (GPU)"]
        end

        subgraph ClusterServices ["Cluster Services"]
            Technitium["Technitium DNS"]
            Gitea["Gitea Local Git"]
            Tailscale["Tailscale Exit Node"]
            Hermes["Hermes AI Agent"]

            subgraph DockerHost ["Docker Server"]
                Traefik["Traefik Proxy Server"]
                Dockhand["Dockhand Docker Management"]
                n8n["n8n Automation Platform"]
                Immich["Immich Photo Library"]
                MCPHoncho["MCP Stack & Honcho AI Memory"]
            end
        end
    end

    %% Ingress & Edge Connections
    Internet --> OpenWrt
    WireGuard --> OpenWrt
    OpenWrt --> Switch

    %% Switch feeds compute nodes
    Switch --> Node1
    Switch --> Node2

    %% Cluster nodes host shared services
    Node1 --> ClusterServices
    Node2 --> ClusterServices

    %% Disaster Recovery
    LocalPBS ===|"Encrypted VPN Tunnel"| PolandPBS

    %% Styling & Color Coding (Dark Theme)
    classDef net fill:#0f2942,stroke:#38bdf8,stroke-width:2px,color:#e2e8f0;
    classDef compute fill:#2e1a05,stroke:#fb923c,stroke-width:2px,color:#e2e8f0;
    classDef storage fill:#260d36,stroke:#c084fc,stroke-width:2px,color:#e2e8f0;
    classDef docker fill:#052e1f,stroke:#34d399,stroke-width:2px,color:#e2e8f0;
    classDef ai fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#e2e8f0;
    classDef ext fill:#1e293b,stroke:#94a3b8,stroke-width:2px,color:#e2e8f0;

    class Internet ext;
    class OpenWrt,Switch,WireGuard,Technitium,Tailscale net;
    class Node1,Node2,ClusterServices compute;
    class LocalPBS,NAS,PolandPBS storage;
    class DockerHost,Traefik,Dockhand,Immich,Gitea docker;
    class AIServer,Hermes,HomeAssistant,n8n,MCPHoncho ai;
```

## Diagram 2: network and trust boundaries

**File:** `assets/diagrams/network-topology.svg`

Include:

- OpenWrt;
- managed switch;
- core, IoT, guest and cluster VLANs;
- Technitium DNS;
- WireGuard remote access;
- permitted and restricted flows.

Use one colour per trust zone. Show the purpose of a boundary rather than the complete firewall rule set.

## Diagram 3: JARVIS and application flow

**File:** `assets/diagrams/jarvis-platform.svg`

Include:

```text
User
 ↓
Hermes
 ├── n8n
 ├── MCP Gateway
 │   ├── Docker MCP
 │   └── custom integrations
 ├── Honcho
 │   └── local models on AI server
 └── cloud reasoning providers

Docker deploy
 ├── Traefik
 ├── Dockhand
 ├── n8n
 ├── Honcho
 ├── MCP stack
 └── Immich core

ubuvault-alpha ── Immich media
AI server ─────── local models and Immich ML
```

This diagram should show responsibility and data flow, not every API call.

## Drawing rules

Create the diagrams in diagrams.net and commit both the source and export:

```text
assets/diagrams/source/homelab-overview.drawio
assets/diagrams/homelab-overview.svg
```

Use the same pattern for all three diagrams.

Recommended colours:

| Layer | Colour |
| --- | --- |
| Networking | Blue |
| Proxmox and compute | Orange |
| Storage and backup | Purple |
| Docker applications | Green |
| Security boundaries | Red |
| JARVIS and automation | Cyan |
| External services | Grey |

Use an explicit background, readable labels and no more detail than the diagram's purpose requires.
