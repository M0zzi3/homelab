# Git-managed Docker deployments

## Problem

Configuration held only in application interfaces or manually maintained Compose files made it difficult to answer what changed, which revision was running and how to return to a previous state.

## Design

Gitea stores focused deployment repositories. Meaningful changes use feature branches and pull requests. Dockhand applies the approved stack to the Docker host, while Traefik provides routes for selected web applications. Runtime secrets remain outside Git.

```text
change
→ feature branch
→ pull request
→ approved configuration
→ Dockhand
→ Docker service
→ health and behaviour checks
→ completion or rollback
```

## Repository principle

A deployment repository contains only what is needed to operate the stack:

```text
service/
├── compose.yml
├── example.env
└── README.md
```

It does not copy the upstream project or store live credentials.

## Verification

A running process is not enough. Depending on the stack, checks include:

- container health and startup logs;
- application health endpoint;
- connectivity through Traefik;
- expected dependent-service behaviour;
- confirmation that the intended revision was deployed.

## Rollback limitation

Returning to an earlier Compose file does not reverse an incompatible database migration. Git configuration, persistent data and application schemas require separate recovery thinking.

## Evidence to add

- one sanitised Compose example;
- a pull-request and deployment screenshot;
- a health-check transcript;
- one failed deployment and rollback exercise.

## What I learned

- Git history improves configuration review but is not a backup for application data;
- smaller deployment repositories are easier to reason about;
- secret recovery must be documented separately;
- pull requests make infrastructure decisions visible even in a one-person lab.

## Professional improvements

A professional implementation would add staging, automated policy checks, managed secrets, signed artifacts, audited approvals and tested migration procedures.

## Skills demonstrated

Git, Gitea, Docker Compose, Dockhand, Traefik, pull requests, secret separation, health verification and rollback planning.
