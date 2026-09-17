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

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart TD
    %% Top: Edge & Gateway Layer
    subgraph Ingress ["Edge & Gateway"]
        Internet(("Public Internet"))
        WireGuard["WireGuard VPN (Remote Admin)"]
        OpenWrt["OpenWrt Firewall / Router"]
        Switch["TP-Link Managed Switch"]

        Internet -->|"WAN / Drop Inbound"| OpenWrt
        WireGuard -->|"Authenticated VPN"| OpenWrt
        OpenWrt <-->|"802.1Q VLAN Trunk"| Switch
    end

    %% 4 VLAN Boxes Side-by-Side Underneath
    subgraph ClusterVLAN ["Cluster VLAN"]
        Corosync["Proxmox Corosync & Migration (No IP Gateway)"]
    end

    subgraph CoreVLAN ["Core VLAN (Trusted & MGMT)"]
        TrustedClients["Trusted Clients (Workstations / Mobile)"]
        ProxmoxMgmt["Proxmox Management (Web UI / SSH)"]
        HA["Home Assistant (Smart Home Hub)"]
        CoreServices["Core Infrastructure (Technitium DNS, Docker, NAS)"]
    end

    subgraph IoTVLAN ["IoT VLAN (Smart Home)"]
        IoTDevices["Smart Devices, Sensors & Plugs"]
    end

    subgraph GuestVLAN ["Guest VLAN (Visitors)"]
        GuestClients["Guest Devices (Isolated)"]
    end

    %% Switch Downlink Connections
    Switch <-->|"Dedicated L2 Ports"| ClusterVLAN
    Switch <-->|"VLAN Trunk (Full Access)"| CoreVLAN
    Switch <-->|"VLAN Trunk (Internet + Filtered)"| IoTVLAN
    Switch <-->|"VLAN Trunk (Internet Only)"| GuestVLAN

    %% Inter-VLAN & Node Policies
    ProxmoxMgmt ===|"Dedicated NICs (Corosync L2)"| Corosync
    TrustedClients -->|"Admin & Control"| IoTDevices
    IoTDevices -->|"Telemetry (HA only)"| HA
    IoTDevices -.->|"Blocked: Dropped by Firewall"| CoreServices
    GuestClients -.->|"Blocked: Zero Internal Access"| CoreVLAN

    %% Styles for 4 VLAN Boxes
    style Ingress fill:#0f172a,stroke:#64748b,stroke-width:2px,color:#e2e8f0;
    style ClusterVLAN fill:#061e33,stroke:#0ea5e9,stroke-width:2px,color:#e2e8f0;
    style CoreVLAN fill:#042116,stroke:#10b981,stroke-width:2px,color:#e2e8f0;
    style IoTVLAN fill:#271403,stroke:#f97316,stroke-width:2px,color:#e2e8f0;
    style GuestVLAN fill:#1d0b2e,stroke:#a855f7,stroke-width:2px,color:#e2e8f0;

    %% Node styling
    classDef edgeNode fill:#1e293b,stroke:#94a3b8,stroke-width:1.5px,color:#f8fafc;
    classDef clusterNode fill:#0c4a6e,stroke:#38bdf8,stroke-width:1.5px,color:#f8fafc;
    classDef coreNode fill:#064e3b,stroke:#34d399,stroke-width:1.5px,color:#f8fafc;
    classDef iotNode fill:#451a03,stroke:#fb923c,stroke-width:1.5px,color:#f8fafc;
    classDef guestNode fill:#3b0764,stroke:#c084fc,stroke-width:1.5px,color:#f8fafc;

    class Internet,WireGuard,OpenWrt,Switch edgeNode;
    class Corosync clusterNode;
    class TrustedClients,ProxmoxMgmt,HA,CoreServices coreNode;
    class IoTDevices iotNode;
    class GuestClients guestNode;
```

Include:

- OpenWrt;
- managed switch;
- core, IoT, guest and cluster VLANs;
- Technitium DNS;
- WireGuard remote access;
- permitted and restricted flows.

Use one colour per trust zone. Show the purpose of a boundary rather than the complete firewall rule set.

## Diagram 3: Subsystem interconnections and application flows

These diagrams detail the functional lifecycles and cross-system data flows that operate across the infrastructure.

### 3.1 AI and automation stack (JARVIS platform)

Shows conversational prompt handling, agent orchestration via Hermes, durable automation in n8n, tool integration through the MCP stack, conversational memory in Honcho, local GPU inference, and cloud fallback.

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart LR
    %% Actors
    User(("User / Voice / Chat")) --> Hermes["Hermes Agent Runtime\n(JARVIS Gateway)"]

    %% Core Orchestration
    Hermes <-->|"Schedules & Triggers"| n8n["n8n Automation Engine"]
    Hermes <-->|"Tool Execution"| MCP["MCP Gateway"]

    subgraph MCPTools ["MCP Stack"]
        DockerMCP["Docker MCP (Container Ops)"]
        GarminMCP["Garmin MCP (Health / Fitness Data)"]
    end
    MCP --> DockerMCP & GarminMCP

    %% Context & Reasoning
    Hermes <-->|"Session Memory"| Honcho["Honcho Memory System"]
    Honcho -->|"Local Embeddings & Summaries"| AIServer["AI Server (GPU / LM Studio)"]
    Hermes -.->|"Complex Reasoning Fallback"| CloudLLM["Cloud LLMs"]

    %% Computer Vision / App ML
    ImmichApp["Immich Core"] -->|"ML Tasks (CLIP & Face Recognition)"| AIServer

    %% Styling
    style MCPTools fill:#120e2e,stroke:#818cf8,stroke-width:1.5px,color:#e2e8f0;
    classDef ai fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#f8fafc;
    classDef client fill:#0f2942,stroke:#38bdf8,stroke-width:2px,color:#f8fafc;
    class Hermes,n8n,MCP,DockerMCP,GarminMCP,Honcho,AIServer,CloudLLM,ImmichApp ai;
    class User client;
```

### 3.2 GitOps and container delivery pipeline

Traces the path from code changes and PR reviews in Gitea through automated Dockhand deployment to the Docker host, reverse-proxy ingress via Traefik, and persistent storage mounts.

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart LR
    %% Ingress & GitOps
    Dev(("Developer")) -->|"Feature Branch & PR"| Gitea["Gitea Source Control"]
    Gitea -->|"Approved Compose Stacks"| Dockhand["Dockhand Deployment Engine"]
    Dockhand -->|"Stack Deploy & Health Check"| DockerHost["Docker Host Containers"]

    %% Ingress Route
    User(("Web / Mobile Clients")) --> Traefik["Traefik Reverse Proxy"]
    Traefik -->|"Routes Ingress"| DockerHost

    %% Storage Persistence
    DockerHost <-->|"Persistent Volumes"| NAS["ubuvault-alpha NAS"]

    %% Styling
    classDef gitops fill:#064e3b,stroke:#34d399,stroke-width:2px,color:#f8fafc;
    classDef actor fill:#1e293b,stroke:#94a3b8,stroke-width:2px,color:#f8fafc;
    classDef storage fill:#260d36,stroke:#c084fc,stroke-width:2px,color:#f8fafc;
    class Gitea,Dockhand,Traefik,DockerHost gitops;
    class Dev,User actor;
    class NAS storage;
```

### 3.3 Central storage architecture (ubuvault-alpha NAS)

Details how the NAS functions as the shared high-capacity storage backbone across virtual machines, containers, and hypervisors.

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart LR
    subgraph NASPools ["ubuvault-alpha NAS (Storage Backbone)"]
        Photos["Immich Media (/photos & /videos)"]
        GitData["Gitea Repositories Storage"]
        ISOPool["Proxmox ISOs & VM Templates"]
        PrivateShares["Private User Shares (SMB / NFS)"]
    end

    %% Consumers
    Immich["Immich Container"] <-->|"Direct Media Mount"| Photos
    Gitea["Gitea LXC"] <-->|"Git Volume Mount"| GitData
    PVE["Proxmox VE Cluster"] <-->|"NFS Shared Storage"| ISOPool
    Clients["Workstations & LAN Clients"] <-->|"SMB / NFS Access"| PrivateShares

    %% Styling
    style NASPools fill:#240c30,stroke:#c084fc,stroke-width:2px,color:#e2e8f0;
    classDef storageNode fill:#3b0764,stroke:#c084fc,stroke-width:1.5px,color:#f8fafc;
    classDef clientNode fill:#0f2942,stroke:#38bdf8,stroke-width:1.5px,color:#f8fafc;
    class Photos,GitData,ISOPool,PrivateShares storageNode;
    class Immich,Gitea,PVE,Clients clientNode;
```

### 3.4 Backup and disaster recovery pipeline

Illustrates the tiered recovery strategy: scheduled deduplicated guest backups to the local Proxmox Backup Server, followed by encrypted remote replication over VPN to Poland PBS.

```mermaid
---
config:
  theme: dark
  layout: elk
  flowchart:
    curve: linear
---
flowchart LR
    subgraph ComputeGuests ["Proxmox Virtual Machines & LXCs"]
        DockerVM["Docker Deploy Host"]
        InfraVMs["Core Infrastructure Guests"]
        AIVM["AI Server"]
        HAServer["Home Assistant"]
    end

    subgraph LocalBackup ["Local Recovery Tier"]
        LocalPBS[("Proxmox Backup Server (Local)")]
        LocalRestore["Fast Local Restore & Deduplication"]
    end

    subgraph OffsiteBackup ["Off-Site Disaster Recovery"]
        PolandPBS[("Proxmox Backup Server (Poland)")]
        RemoteRetention["Geographic Separation (Off-Site)"]
    end

    %% Flows
    ComputeGuests -->|"Scheduled Deduplicated Snapshots"| LocalPBS
    LocalPBS --- LocalRestore
    LocalPBS ===|"Encrypted VPN Tunnel (Remote Sync)"| PolandPBS
    PolandPBS --- RemoteRetention

    %% Styling
    style ComputeGuests fill:#052e1f,stroke:#34d399,stroke-width:1.5px,color:#e2e8f0;
    style LocalBackup fill:#2b0b1b,stroke:#f43f5e,stroke-width:2px,color:#e2e8f0;
    style OffsiteBackup fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#e2e8f0;
    classDef nodeStyle fill:#1e293b,stroke:#94a3b8,stroke-width:1.5px,color:#f8fafc;
    class DockerVM,InfraVMs,AIVM,HAServer,LocalPBS,LocalRestore,PolandPBS,RemoteRetention nodeStyle;
```

## Diagram standards

Diagrams are maintained as native, version-controlled Mermaid diagrams directly within the repository markdown pages:

- **Theme:** Dark theme with high-contrast functional color-coding.
- **Layout Engine:** ELK (`layout: elk`) with linear orthogonal edge routing (`curve: linear`).
- **Boundaries:** Clear conceptual trust and execution zones with explicit direction of data and authority flow.
