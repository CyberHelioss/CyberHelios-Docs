# CyberHelios Documentation

Central repository for MeshGate architecture, decisions, specifications, and operational guidelines.

## Quick Navigation

- **[Architecture Overview](./ARCHITECTURE.md)** — System design, components, and data flow
- **[Architecture Decision Records (ADRs)](./adr/)** — Rationale behind technical decisions
- **[API Specifications](./api/)** — OpenAPI contracts between services
- **[Deployment Guide](./DEPLOYMENT.md)** — Infrastructure setup (Proxmox, Docker, LXC)
- **[Security Model](./SECURITY.md)** — Authentication, authorization, threat model
- **[Development Setup](./DEVELOPMENT.md)** — Local environment configuration
- **[Roadmap](./ROADMAP.md)** — 4-phase delivery plan with milestones
- **[Contributing](./CONTRIBUTING.md)** — Code standards, PR process, testing

## Project Overview

**MeshGate** is a self-hosted edge networking platform that solves the challenge of securely exposing services from dynamic networks. It combines:

- **Secure Public Gateway** — reverse proxy with HTTPS termination
- **Dynamic DNS Management** — automatic IP tracking and DNS updates
- **Encrypted Mesh Networking** — private peer-to-peer communication
- **Central Control Plane** — orchestration and policy management
- **Dashboard UI** — intuitive system administration
- **Infrastructure Automation** — deployment and lifecycle management

## Core Repositories

### Product Repositories
| Repo | Purpose | Tech |
|------|---------|------|
| [MeshGate-Gateway](https://github.com/CyberHelioss/MeshGate-Gateway) | Public-facing reverse proxy | Caddy/Nginx + Go |
| [MeshGate-DDNS](https://github.com/CyberHelioss/MeshGate-DDNS) | Dynamic DNS service | Go |
| [MeshGate-DDNS-Worker](https://github.com/CyberHelioss/MeshGate-DDNS-Worker) | DDNS job execution | Go |
| [MeshGate-Mesh](https://github.com/CyberHelioss/MeshGate-Mesh) | Encrypted mesh networking | Go |
| [MeshGate-ControlPlane](https://github.com/CyberHelioss/MeshGate-ControlPlain) | Central orchestration API | Go + PostgreSQL |
| [MeshGate-Dashboard](https://github.com/CyberHelioss/MeshGate-Dashboard) | Web administration UI | React + Next.js |

### Shared Repositories
| Repo | Purpose | Tech |
|------|---------|------|
| [CyberHelios-Shared-Auth](https://github.com/CyberHelioss/CyberHelios-Shared-Auth) | JWT + RBAC library | Go |
| [CyberHelios-Shared-Monitoring](https://github.com/CyberHelioss/CyberHelios-Shared-Monitoring) | Metrics + observability | Go |
| [CyberHelios-infra](https://github.com/CyberHelioss/CyberHelios-infra) | Infrastructure as Code | Terraform + Ansible |

## Development Phases

### Phase 1: Foundation (Weeks 1-4)
- [ ] Shared libraries scaffolding
- [ ] ControlPlane API contracts
- [ ] CI/CD pipelines
- [ ] Development environment setup

### Phase 2: Core Services (Weeks 5-12)
- [ ] ControlPlane implementation
- [ ] Gateway reverse proxy
- [ ] Mesh networking layer
- [ ] DDNS service

### Phase 3: Integration (Weeks 13-20)
- [ ] Service interconnection testing
- [ ] Dashboard frontend development
- [ ] Infrastructure automation

### Phase 4: Polish & Release (Weeks 21+)
- [ ] Security hardening
- [ ] Performance optimization
- [ ] User documentation
- [ ] v1.0 release

## Technology Stack

**Backend:** Go 1.21+  
**Database:** PostgreSQL 14+  
**Cache/Jobs:** Redis (optional)  
**Frontend:** React 18 + Next.js 14 + Tailwind CSS  
**Reverse Proxy:** Caddy or Nginx  
**Monitoring:** Prometheus + Grafana + Loki  
**Infrastructure:** Docker + Terraform + Ansible  
**CI/CD:** GitHub Actions  
**Authentication:** JWT + RBAC  

## Key Principles

1. **Modularity** — Each repo has a single responsibility
2. **Security First** — Auth, encryption, and input validation by default
3. **Observability** — Metrics, logs, and tracing built-in
4. **OpenAPI Contracts** — Clear, versioned API boundaries
5. **Asynchronous Documentation** — Decisions live in code, not Slack
6. **Automated Quality Gates** — Tests, linting, security scans on every PR

## Getting Started

**For Developers:**
See [Development Setup](./DEVELOPMENT.md) for local environment configuration.

**For Operators:**
See [Deployment Guide](./DEPLOYMENT.md) for production infrastructure setup.

**For Contributors:**
See [Contributing](./CONTRIBUTING.md) for code standards and PR process.

## Important Links

- **Organization:** [github.com/CyberHelioss](https://github.com/CyberHelioss)
- **Issues:** See individual repository issue trackers
- **Discussions:** Use GitHub Discussions in each repo for design proposals
- **License:** All projects are GPL-3.0

## Status

🟡 **Pre-Alpha** — Foundation phase in progress. APIs and implementation details subject to change. Not production-ready.

---

**Last Updated:** June 5, 2026  
**Maintainers:** See OWNERS.md in individual repositories
