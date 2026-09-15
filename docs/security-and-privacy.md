# Security and privacy

## Purpose

This repository is public. It must demonstrate engineering work without exposing information that would make the live environment easier to target.

## Public material

The following may be published after review:

- conceptual architecture diagrams;
- fictionalised hostnames and network ranges;
- technology choices and design rationale;
- selected configuration fragments using placeholders;
- generic verification commands;
- screenshots with identifying fields removed;
- incident summaries that do not reveal access paths;
- backup and recovery concepts without real destinations or credentials.

## Private material

The following must remain in private systems:

- passwords, tokens, API keys and OAuth material;
- `.env` files and application credential exports;
- private keys, certificates and certificate-authority material;
- full WireGuard, Tailscale or remote-access configuration;
- public addresses and port-forwarding rules;
- real MAC addresses, device serial numbers and account identifiers;
- complete firewall exports;
- unredacted Home Assistant configuration or entity history;
- n8n credentials and production execution payloads;
- backup credentials, encryption keys and exact remote destinations;
- screenshots containing emails, usernames, tokens or internal browser sessions.

Private address space is not automatically sensitive, but exact topology still has little portfolio value. Public diagrams therefore use roles and trust zones instead of a live address plan.

## Threats considered

### Accidental secret commit

A credential enters Git history through an environment file, copied command or workflow export.

Controls:

- deny common secret and environment-file patterns in `.gitignore`;
- enable Gitea and GitHub secret scanning where available;
- review diffs before every public push;
- rotate any exposed credential rather than relying on history rewriting alone.

### Sensitive screenshot

An otherwise useful dashboard image exposes an address, email, token, hostname or client record.

Controls:

- capture only the required window or panel;
- redact before adding the image to Git;
- inspect the final image at full resolution;
- remove metadata when appropriate.

### Overly complete configuration example

A sanitised file still reveals the exact access model or can be combined with other public information.

Controls:

- publish the smallest fragment that proves the engineering point;
- replace names and addresses consistently;
- omit unrelated sections;
- describe intent in prose instead of publishing full exports.

## Pre-publication checklist

Before merging a change that will be mirrored publicly:

- [ ] No credentials, tokens, private keys or secret values
- [ ] No `.env`, VPN, certificate or application credential files
- [ ] No real public addresses or port-forwarding rules
- [ ] No MAC addresses, serial numbers or personal identifiers
- [ ] No screenshots with sensitive fields
- [ ] Internal names and ranges are fictionalised where appropriate
- [ ] Configuration examples use obvious placeholders
- [ ] Links do not point to private administrative interfaces
- [ ] The diff has been reviewed from the perspective of an external reader

## If sensitive data is published

1. Remove public access to the affected material.
2. Rotate or revoke the exposed secret immediately.
3. Check logs for use of the exposed credential.
4. Remove the material from the current tree and, where necessary, Git history.
5. Document the incident privately and improve the publication check that failed.

Deleting a file in a later commit does not remove it from earlier Git history.
