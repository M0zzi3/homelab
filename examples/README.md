# Sanitised examples

This directory will hold small configuration and automation examples that support the case studies.

Each example must:

- use fictional hostnames, addresses and identifiers;
- use placeholders for every credential or secret value;
- contain only the fragment needed to demonstrate the technical point;
- include a short explanation and verification step;
- be checked against the publication policy before merge.

Planned examples:

- a minimal Docker Compose deployment;
- a post-deployment health-check script;
- an n8n workflow export using sample data;
- a DNS or network-policy fragment using fictional ranges;
- a lightweight secret-pattern check for CI.

The private deployment repositories remain the source of truth for the live environment.
