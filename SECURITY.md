# Security policy

This repository is a sanitised portfolio view of a private homelab.

## Reporting a problem

If you believe a file exposes a credential, private key, personal identifier or operational access detail, use Gitea's private contact channel for the repository owner. Do not include the sensitive value in a public issue.

## Scope

Security reports are useful for:

- accidentally committed secrets;
- public configuration that exposes a meaningful access path;
- scripts in `examples/` with an unsafe default;
- documentation that identifies private infrastructure more precisely than intended.

The live homelab and its private services are not offered as a public security-testing target. Do not scan or attempt to access systems referenced by this repository.

See [Security and privacy](docs/security-and-privacy.md) for the publication policy.
