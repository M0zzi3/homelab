# Operations

This page describes how the homelab is changed, accessed and recovered. It records the operating model rather than every command used by every service.

## Deployment and CI/CD

The Docker platform uses focused deployment repositories in Gitea:

```text
deploy/dockhand
deploy/honcho
deploy/immich
deploy/mcp-stack
deploy/n8n
deploy/traefik
```

The intended change path is:

```text
feature branch
→ pull request
→ approved main revision
→ Dockhand deployment
→ container and application checks
→ completion or rollback
```

This is GitOps-style delivery, not a claim that every deployment is fully autonomous. Manual approval remains useful for stateful or high-impact services.

Traefik is the common HTTP/HTTPS edge for selected Docker applications. Routing configuration should make the service destination, authentication requirement, exposure scope and certificate ownership clear.

A deployment is not considered successful merely because a container is running. Verification can include container health, startup logs, application endpoints, reverse-proxy routing and dependent service behaviour.

See the [GitOps and container delivery pipeline](architecture.md#32-gitops-and-container-delivery-pipeline).

## Backup and recovery

The recovery model has several layers (see the [backup and disaster recovery pipeline](architecture.md#34-backup-and-disaster-recovery-pipeline)):

- local Proxmox backups for protected VMs and LXCs;
- a second PBS in Poland connected through VPN;
- Git history for non-secret deployment configuration;
- application-specific database and file backups;
- recovery notes for state that cannot be reconstructed automatically.

The final backup evidence should include:

- schedules and retention;
- PBS verification jobs;
- off-site sync ownership and failure behaviour;
- encryption-key recovery;
- at least one measured restore.

A Compose rollback cannot reverse an incompatible database migration. Container configuration and application data therefore have separate recovery procedures.

## Remote access and network operations

OpenWrt provides routing, firewall policy and DHCP. The managed switch carries the core, IoT, guest and Proxmox VLANs. Technitium provides internal DNS. WireGuard provides authenticated remote access.

Administrative interfaces are not intentionally exposed directly to the public internet. The public portfolio omits peer keys, endpoints, live addresses and complete firewall rules.

Useful network verification includes:

- client DNS resolution;
- permitted and denied inter-VLAN paths;
- switch trunk/access configuration;
- remote access to approved services;
- loss-of-DNS and loss-of-VPN behaviour.

## JARVIS operations

JARVIS is distributed across several components (see the [AI and automation stack diagram](architecture.md#31-ai-and-automation-stack-jarvis-platform)):

| Component | Responsibility |
| --- | --- |
| Hermes | User interaction, tool selection and model routing |
| n8n | Durable schedules and deterministic workflows |
| MCP Gateway | Common interface for authorised integrations |
| Docker MCP | Restricted Docker operations |
| Honcho | Durable conversational context |
| Local AI server | Lightweight memory and embedding workloads |
| Cloud providers | Hard reasoning when local capacity is insufficient |

Recurring workflows stay in n8n. Hermes is called asynchronously when judgement is needed. Raw domain data remains in the appropriate system of record rather than being copied indiscriminately into conversational memory.

## Incident approach

1. Confirm the affected service and user impact.
2. Inspect live state, dependencies and recent changes.
3. Restore the last known-good configuration when the incident is urgent.
4. Verify useful behaviour after recovery.
5. Investigate the cleaner long-term fix once service is stable.
6. Document the failure, decision and evidence.

Examples suitable for future public write-ups include Docker disk pressure, DNS authority migration and local-model compatibility failures.

## Evidence still needed

- verified Proxmox RAM and guest allocation table;
- list of actual HA-managed guests;
- PBS sync direction and restore timing;
- NAS filesystem and redundancy model;
- selected screenshots with sensitive fields removed;
- sanitised configuration fragments and health checks.
