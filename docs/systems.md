# System catalogue

This page explains the major VMs, LXCs and infrastructure systems. It is deliberately concise; changing container IDs, image tags and live addresses belong in private operational documentation.

## Infrastructure and platform systems

| System | Runtime role | Main responsibility |
| --- | --- | --- |
| Technitium DNS | Infrastructure guest | Internal DNS and custom private zones |
| AI server | GPU-backed VM | Local models, embeddings and Immich ML |
| Docker deploy | Application VM | Shared container platform and delivery edge |
| Proxmox Backup Server | Backup system | Local guest recovery and off-site replication |
| Gitea | Source-control LXC | Repositories, pull requests and public mirrors |
| Hermes | Agent LXC | JARVIS runtime, dashboard, gateway and tools |
| ubuvault-alpha | Storage system | NAS and persistent media storage |
| Home Assistant | Automation guest | Smart-home control and integrations |

## Technitium DNS

OpenWrt provides DHCP and advertises Technitium as the resolver. Technitium hosts the internal DNS zone used by infrastructure and application services. Important systems use explicit records rather than depending only on changing client leases.

**Evidence to add:** a sanitised zone layout, representative lookup tests and the DNS migration decision.

## AI server

The AI server is a GPU-backed VM. It provides LM Studio model serving for Honcho and selected JARVIS workloads, and it runs the machine-learning side of Immich.

The persistent local workload uses a small Qwen language model and Nomic embeddings. More demanding reasoning remains on the main JARVIS provider path so the GPU retains headroom for other work.

**Evidence to add:** verified VM allocation, GPU observations under competing workloads and an end-to-end Honcho request.

## Docker deploy

This VM is the central container host. A live inventory during the documentation work confirmed:

- Traefik and Dockhand;
- n8n;
- Docker MCP and MCP Gateway;
- Honcho API, deriver, PostgreSQL/pgvector and Redis;
- Immich server, PostgreSQL and Valkey/Redis.

Non-secret stack configuration is stored in focused Gitea repositories. Traefik provides stable application routes, while internal databases and Docker administration remain private.

**Evidence to add:** one sanitised deployment, health verification and rollback example.

## Proxmox Backup Server

The local PBS stores backups of protected Proxmox guests. A second PBS in Poland provides geographic separation over a private VPN connection.

The exact replication direction still needs verification. PBS sync is commonly initiated by the destination as a pull, even when the source is described as sending backups.

**Evidence to add:** sanitised retention policy, verification result, remote sync relationship and one measured restore.

## Gitea

Gitea is the working source of truth for school projects, deployment repositories and this portfolio. Changes use branches and pull requests. Selected repositories are mirrored to GitHub for public presentation.

Gitea also provides configuration to Dockhand and selected repository events to n8n. JARVIS uses a dedicated contributor account with repository-specific permissions.

**Evidence to add:** repository organisation, a pull-request example and mirror verification.

## Hermes

Hermes hosts JARVIS's conversation runtime, gateway, dashboard, skills and tool orchestration. It connects to n8n, MCP services, Home Assistant, Honcho and model providers.

The public portfolio does not include credentials, personal memory, session data or capability configuration.

**Evidence to add:** a sanitised asynchronous run and integration flow.

## ubuvault-alpha

`ubuvault-alpha` is the NAS and stores persistent file data used by services such as Immich. Application files and databases are treated as separate recovery concerns: protecting the photo library alone does not recreate the complete Immich application state.

See the [central storage architecture diagram](architecture.md#33-central-storage-architecture-ubuvault-alpha-nas) for consumer mount allocations.

**Evidence to add:** verified storage layout, redundancy model, capacity monitoring and a representative restore.

## Home Assistant

Home Assistant manages smart-home devices, MQTT/local integrations and deterministic automations. It runs separately from the shared Docker application host. IoT devices occupy their own network zone, with only required flows crossing into trusted services.

JARVIS and n8n can interact with selected entities, but frequent presence or motion events are not allowed to create unnecessary AI workloads.

**Evidence to add:** one sanitised automation, the IoT-to-Home-Assistant network flow and a troubleshooting example.

## Immich data path

Immich spans several systems:

```text
Trusted client → Traefik → Immich server
Immich server → PostgreSQL and cache
Immich server → photo/video library on ubuvault-alpha
Immich server → machine learning on AI server
```

Recovery must account for the media library, PostgreSQL metadata and deployment configuration separately.
