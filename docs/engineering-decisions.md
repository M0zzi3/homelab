# Engineering decisions

This document records why major architectural choices were made. It is intentionally shorter than a formal Architecture Decision Record collection; individual decisions can be promoted to separate ADR files when they need more history.

## Decision summary

| Decision | Status | Reason |
| --- | --- | --- |
| Use Proxmox VE as the virtualisation layer | Adopted | Supports VMs, LXCs, clustering and integrated backup workflows on available hardware |
| Separate cluster communication from general services | Adopted | Reduces contention and limits unnecessary exposure of cluster traffic |
| Keep important configuration in Git | Adopted | Provides reviewable history, rollback points and a clear source for deployments |
| Keep secrets outside deployment repositories | Adopted | Reduces accidental credential exposure and separates intent from runtime identity |
| Place GPU-dependent services on a dedicated AI host | Adopted | Allows explicit capacity planning and avoids assigning GPU workloads to every node |
| Use local models for persistent lightweight workloads | Adopted | Preserves privacy and predictable cost for memory and embedding tasks |
| Use stronger cloud models selectively | Adopted | Avoids forcing a small local model to handle tasks beyond its reliable capability |
| Publish only a sanitised portfolio view | Adopted | Demonstrates engineering work without publishing operational access details |

## Proxmox over a single Docker host

A single Docker machine would have been simpler, but it would not provide the same environment for learning workload isolation, VM and LXC trade-offs, cluster networking, backup and recovery.

Proxmox became the platform layer. Docker remains an application delivery mechanism inside selected guests rather than replacing the hypervisor.

### Trade-off

The design has more moving parts and requires additional patching, backup and monitoring discipline. That cost is acceptable because infrastructure operations are one of the main learning goals.

## Dedicated cluster network

Cluster communication is kept separate from general application and client traffic.

### Why

- cluster traffic has different reliability and latency requirements;
- application or guest traffic should not unnecessarily interfere with cluster communication;
- the boundary makes troubleshooting and firewall intent clearer.

### Trade-off

The design requires managed switching, VLAN configuration and consistent node networking.

## Gitea and deployment repositories

Gitea is the internal source-control service. Deployment repositories describe non-secret application configuration, while Dockhand applies selected stacks to Docker hosts.

### Why

- infrastructure changes gain history and review;
- a known-good revision provides a rollback target;
- configuration can be inspected without relying on a running application UI;
- self-hosting keeps core deployment control available on the local network.

### Trade-off

Git history does not automatically protect application data. Backups and application-specific exports remain separate concerns.

## Runtime secrets outside Git

Environment files containing credentials are not committed to deployment repositories. Runtime systems provide secret values separately.

### Why

The repository should express desired configuration without becoming a credential store.

### Trade-off

A new deployment requires both the repository and the separately managed secret material. Recovery documentation must cover both.

## Split local and cloud AI workloads

Small local models handle persistent, privacy-sensitive or predictable tasks. More demanding reasoning can be routed to external providers.

### Why

The available GPU must also support other workloads. Running the largest possible model continuously would reduce reliability and leave no capacity for experiments or media processing.

### Trade-off

Routing introduces more operational paths and requires clear handling for provider failure, privacy boundaries and cost.

## Public portfolio separated from operational documentation

This repository explains the architecture but does not mirror the live deployment repositories or the full network workbook.

### Why

A recruiter needs evidence of technical judgement. They do not need addresses, credentials or every service-specific setting.

### Trade-off

Public examples require maintenance and sanitisation when the live system changes.
