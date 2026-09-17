# Proxmox platform

## Context

I wanted a persistent environment for learning Linux administration, networking and service operations outside isolated university labs. The platform had to host infrastructure, applications, home automation and AI workloads on limited residential hardware.

## Design

Proxmox VE is the virtualisation layer. It runs a mixture of VMs and LXCs, with application containers inside selected guests. Cluster communication uses a dedicated VLAN, and a quorum device supports the two-node design.

Workloads are separated by responsibility:

- infrastructure services such as DNS and source control;
- a central Docker application host;
- a GPU-backed AI server;
- Home Assistant and Hermes;
- NAS and backup systems.

## Why this design

A single Docker machine would be simpler, but it would provide less experience with VM/LXC placement, cluster networking, backup and recovery. Proxmox adds operational work, but that work is part of the purpose of the lab.

## High availability and capacity

Cluster membership is not the same as automatic guest HA. Before this page claims high availability, the public evidence must confirm:

- which guests are managed by Proxmox HA;
- their storage and placement dependencies;
- node RAM and guest allocation;
- capacity available when one node fails;
- tested quorum and restart behaviour.

The Ceph/HA environment from my university Practice Enterprise project is separate and is not presented as the storage architecture of this homelab.

## Verification to add

- sanitised node and guest allocation table;
- backup and restore result;
- network and quorum checks;
- one measured migration or recovery exercise.

## What I learned

- placement is an operational decision, not only a resource calculation;
- recovery capacity matters before enabling automated HA;
- hypervisor backups do not replace application-aware recovery;
- clear names and documentation reduce troubleshooting time.

## Professional improvements

With enterprise requirements I would add redundant switching and power, central authentication, wider monitoring, formal recovery objectives and scheduled restore tests.

## Skills demonstrated

Proxmox VE, KVM, LXC, Linux, VLANs, cluster networking, quorum, capacity planning, backup integration and failure-domain analysis.
