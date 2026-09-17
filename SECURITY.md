# Security policy

This repository is a sanitised portfolio view of a private homelab.

## Public boundary

The repository may include conceptual diagrams, design decisions, fictionalised examples and selected verification evidence.

It must not include:

- passwords, tokens, API keys or OAuth material;
- private keys, certificates or VPN configuration;
- public addresses or port-forwarding rules;
- MAC addresses, serial numbers or personal identifiers;
- full firewall and application exports;
- backup credentials or encryption material;
- screenshots containing private data or administrative sessions.

## Reporting an issue

If a file appears to expose sensitive information, contact the repository owner privately. Do not repeat the value in a public issue.

The systems described here are not offered as public security-testing targets.

## Before publishing

- review the complete diff;
- inspect images at full resolution;
- replace credentials with obvious placeholders;
- confirm links do not target private administration interfaces;
- publish only the smallest configuration fragment needed to prove the point.

If a credential is exposed, revoke or rotate it immediately. Deleting it in a later commit does not remove it from Git history.
