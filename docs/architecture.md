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
flowchart LR
    n1(("Internet")) --> n2["OpenWRT Firewall/Router"]
    n3["tp-link Switch"] --> n4["Proxmox Cluster"] & n5["Proxmox Node 2 endurance"]
    n2 --> n3 & n16["WireGuard VPN"]
    n4 --> n6["Proxmox Backup Server"] & n7["Home Assistant"] & n8["NAS Server"] & n10["High Availability Services"]
    n5 --> n9["AI Server"] & n10
    n10 --> n11["Docker Server"] & n12["Technitium DNS"] & n13["Tailscale VPN Exit Node"] & n14["Herme AI Agent"] & n15["Gitea Local Git Server"]
    n11 --> n17["Immich Photo Library"] & n18["Traefik Proxy Sever"] & n19["Dockhand Docker Managment"] & n20["n8n Aumtomation Platform"] & n21["MCP Stack"] & n25["Honcho AI Memroy System"]n11 --> n17["Immich Photo Library"] & n18["Traefik Proxy Sever"] & n19["Dockhand Docker Managment"] & n20["n8n Aumtomation Platform"] & n21["MCP Stack"] & n25["Honcho AI Memroy System"]
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
