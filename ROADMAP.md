# MeshGate Development Roadmap

## Phase-Based Delivery Plan (24 Weeks)

Structured timeline with clear dependencies, milestones, and success criteria.

---

## Phase 1: Foundation (Weeks 1-4)

**Goal:** Establish core infrastructure, shared libraries, and API contracts  
**Success Criteria:** All repos have scaffolding, CI/CD pipelines working, team can work in parallel

### Week 1: Scaffolding & Setup
**Owner:** Project Lead + Backend Lead

- [ ] Initialize Go project structure for all backend repos
  - `cmd/`, `internal/`, `pkg/` directory structure
  - `go.mod` with dependency management
  - Makefile with `lint`, `test`, `build`, `run` targets
  
- [ ] Initialize React/Next.js project for Dashboard
  - Create-next-app with Tailwind CSS
  - Basic routing and layout structure
  - TypeScript configuration
  
- [ ] Create shared Dockerfile and docker-compose.yml for local development
  - PostgreSQL 14 container
  - Redis container (optional)
  - Volume mounts for code
  
- [ ] Set up GitHub Actions CI/CD template
  - Lint checks (golangci-lint, eslint)
  - Test runs with coverage reports
  - Build and push Docker images
  
**Deliverables:**
- All 9 repos have language-appropriate scaffolding
- CI/CD pipelines run on every PR
- `docker-compose up` spins up dev environment

### Week 2: Shared Libraries
**Owner:** Backend Lead

**CyberHelios-Shared-Auth:**
- [ ] JWT token generation and validation
- [ ] RBAC role model (admin, operator, read-only)
- [ ] Auth middleware for Go services
- [ ] Password hashing (bcrypt)
- [ ] Session token management
- [ ] Unit tests (>80% coverage)

**CyberHelios-Shared-Monitoring:**
- [ ] Prometheus metrics collector
- [ ] Structured logging (JSON format)
- [ ] Health check endpoint handler
- [ ] Request tracing middleware
- [ ] Unit tests (>80% coverage)

**Deliverables:**
- Both libraries published as Go modules
- Example usage in README
- Integration tests with mock services

### Week 3: Control Plane API Contracts
**Owner:** Backend Lead + Networking Lead

**OpenAPI Specification:**
- [ ] Document all REST endpoints (see `/api/control-plane-openapi.yaml`)
- [ ] Define request/response schemas
- [ ] Define error codes and status codes
- [ ] Generate mock server from spec (Prism or similar)

**Key Endpoints:**
```
Authentication:
  POST   /api/v1/auth/login
  POST   /api/v1/auth/logout
  POST   /api/v1/auth/refresh

Nodes:
  POST   /api/v1/nodes
  GET    /api/v1/nodes
  GET    /api/v1/nodes/{id}
  PUT    /api/v1/nodes/{id}
  DELETE /api/v1/nodes/{id}

Peers (Mesh):
  GET    /api/v1/peers
  POST   /api/v1/peers/{nodeId}/discover

Jobs:
  POST   /api/v1/jobs
  GET    /api/v1/jobs
  GET    /api/v1/jobs/{id}
  PATCH  /api/v1/jobs/{id}

Configuration:
  GET    /api/v1/config
  PATCH  /api/v1/config

Certificates:
  GET    /api/v1/certs
  POST   /api/v1/certs

DNS:
  GET    /api/v1/dns/records
  POST   /api/v1/dns/records
```

**Deliverables:**
- OpenAPI 3.0 specification (YAML)
- Mock server for frontend/testing
- API documentation (rendered HTML)

### Week 4: CI/CD & Collaboration Setup
**Owner:** Infrastructure Lead

- [ ] GitHub branch protection rules (all repos)
  - Require 2 PRs for critical changes
  - Require CI tests to pass
  - Require CODEOWNERS review
  
- [ ] GitHub Projects v2 setup
  - Create 4 epics (Phase 1-4)
  - Create 2-week sprint boards
  - Link issues to epics
  
- [ ] Dependabot configuration
  - Automated dependency updates
  - Security alerts enabled
  
- [ ] OWNERS.md template created
  - Assign maintainers per repo
  
**Deliverables:**
- All repos have branch protection
- GitHub Projects v2 configured with Phase 1 epics
- First sprint board ready
- Team can safely collaborate

---

## Phase 2: Core Services (Weeks 5-12)

**Goal:** Implement core backend services with API contracts locked in  
**Success Criteria:** Services can communicate, basic CRUD operations work, 80%+ test coverage

### Week 5-6: Control Plane (Foundation)
**Owner:** Backend Lead

**MeshGate-ControlPlane:**
- [ ] Set up Go project with Echo/Gin framework
- [ ] PostgreSQL schema design and migrations
- [ ] GORM models for nodes, peers, jobs, config
- [ ] Integrate CyberHelios-Shared-Auth middleware
- [ ] Implement Auth endpoints (login, refresh, logout)
- [ ] Integration tests for auth flow

**Deliverables:**
- Working API on localhost:8080
- PostgreSQL schema versioned
- Auth endpoints testable via curl/Postman

### Week 7: Control Plane (Core APIs)
**Owner:** Backend Lead

- [ ] Node registration and CRUD
- [ ] Peer discovery logic
- [ ] Configuration management endpoints
- [ ] Certificate storage (ACME integration planned for Phase 3)
- [ ] Job scheduling (CRUD operations)
- [ ] Unit and integration tests

**Deliverables:**
- All core CRUD endpoints working
- Swagger/OpenAPI docs generated
- Test coverage >80%

### Week 8: Shared Monitoring Integration
**Owner:** Backend Lead

- [ ] Integrate CyberHelios-Shared-Monitoring into ControlPlane
- [ ] Prometheus metrics endpoints
- [ ] JSON structured logging to stdout
- [ ] Health check endpoint
- [ ] Set up local Prometheus + Grafana in docker-compose

**Deliverables:**
- `GET /metrics` returns Prometheus metrics
- Logs in JSON format
- `GET /health` returns service health
- Grafana dashboard shows ControlPlane metrics

### Week 9-10: Gateway (Reverse Proxy)
**Owner:** Networking Lead

**MeshGate-Gateway:**
- [ ] Use Caddy or Nginx with Go plugin for custom logic
- [ ] HTTPS termination (test cert for now, ACME in Phase 3)
- [ ] Hostname-based routing to internal services
- [ ] Reverse proxy to ControlPlane and services
- [ ] Rate limiting (token bucket algorithm)
- [ ] Security headers (HSTS, CSP, X-Frame-Options)
- [ ] Request/response logging

**Deliverables:**
- Gateway routes requests to mock backends
- HTTPS works with self-signed cert
- Rate limiting enforced
- Logs all requests

### Week 11: Mesh Networking (Foundation)
**Owner:** Networking Lead

**MeshGate-Mesh:**
- [ ] Peer enrollment logic (call Control Plane)
- [ ] WireGuard tunnel setup
- [ ] Basic peer-to-peer communication
- [ ] Health check to peers
- [ ] Integration with CyberHelios-Shared-Auth
- [ ] Mesh node agent binary

**Deliverables:**
- Mesh agents can register and authenticate
- Tunnels established between peers
- Peer health checks working
- Local network communication tests pass

### Week 12: DDNS Service (Foundation)
**Owner:** Networking Lead

**MeshGate-DDNS:**
- [ ] DNS provider abstraction (interface for multiple providers)
- [ ] Route53 provider implementation
- [ ] Cloudflare provider implementation
- [ ] IPv4/IPv6 detection
- [ ] Change detection logic
- [ ] Retry with exponential backoff
- [ ] Unit tests for each provider

**Deliverables:**
- DDNS service can update DNS records
- Multiple providers supported
- Retry logic tested
- Dry-run mode for testing

---

## Phase 3: Integration (Weeks 13-20)

**Goal:** Connect all services, end-to-end testing, Dashboard UI  
**Success Criteria:** Full system works in docker-compose, Dashboard shows live data

### Weeks 13-14: Gateway ↔ Control Plane Integration
**Owner:** Networking Lead

- [ ] Gateway calls ControlPlane API for routing rules
- [ ] Gateway caches routes with TTL
- [ ] ControlPlane pushes route updates to Gateway via WebSocket
- [ ] End-to-end HTTPS routing tests
- [ ] Load testing (10k+ req/sec)

**Deliverables:**
- External request flows through Gateway → ControlPlane → Service
- Route updates propagate in <1s

### Weeks 15-16: Mesh ↔ Control Plane Integration
**Owner:** Networking Lead

- [ ] Mesh agents poll ControlPlane for peer list updates
- [ ] ControlPlane sends ACL rules to mesh agents
- [ ] Mesh tunnel establishment and monitoring
- [ ] Peer discovery tests
- [ ] ACL enforcement tests

**Deliverables:**
- Mesh nodes self-organize
- Private communication between nodes works
- ACLs enforced correctly

### Weeks 17-18: DDNS Integration
**Owner:** Networking Lead

**MeshGate-DDNS-Worker:**
- [ ] Worker polls ControlPlane for DDNS jobs
- [ ] Worker calls DDNS service to update records
- [ ] Worker reports status back to ControlPlane
- [ ] Handles IP change detection and updates
- [ ] Integrated tests with mock DNS provider

**Deliverables:**
- Worker can execute DDNS jobs
- Job status tracked in ControlPlane
- Dashboard shows DDNS status

### Weeks 19-20: Dashboard Frontend
**Owner:** Frontend Lead

**MeshGate-Dashboard:**
- [ ] Authentication flow (login → JWT token)
- [ ] Node/peer visualization (table + basic graph)
- [ ] Real-time updates via WebSocket
- [ ] Configuration editor (JSON editor for now)
- [ ] Job history and logs
- [ ] DNS status and history
- [ ] System health overview

**Deliverables:**
- Dashboard accessible at localhost:3000
- Login works with Control Plane credentials
- Shows live node status
- Responsive design

---

## Phase 4: Polish & Release (Weeks 21+)

**Goal:** Production readiness, security hardening, v1.0 release  
**Success Criteria:** Publicly available, 500+ stars, production deployable

### Weeks 21-22: Security Hardening
**Owner:** Security Lead

- [ ] Security audit of all services
- [ ] OWASP Top 10 checks
- [ ] Input validation on all APIs
- [ ] SQL injection prevention (use parameterized queries)
- [ ] XSS protection in Dashboard
- [ ] CSRF tokens
- [ ] Rate limiting on auth endpoints
- [ ] Secrets management (environment-based)
- [ ] TLS certificate validation

**Deliverables:**
- Security audit report
- All findings remediated
- Penetration testing (contracted out)

### Weeks 23-24: Performance & Optimization
**Owner:** Infrastructure Lead

- [ ] Load testing (10k+ req/sec on Gateway)
- [ ] Database query optimization
- [ ] Connection pooling (PgBouncer)
- [ ] Caching strategy (Redis)
- [ ] Mesh bandwidth optimization
- [ ] Memory profiling and optimization
- [ ] Container resource limits tuned

**Deliverables:**
- Benchmark results documented
- Scalability guidelines published
- Resource limits defined

### Week 25+: Release & Documentation

- [ ] v1.0 tag created
- [ ] Release notes published
- [ ] Docker images pushed to Docker Hub
- [ ] Helm charts (optional, Phase 2 feature)
- [ ] Blog post / announcement
- [ ] YouTube demo video
- [ ] Hacker News / Reddit posts
- [ ] Community outreach

---

## Parallel Work Streams

### Development
- Phase 2 backend development happens in parallel
- Gateway, Mesh, DDNS can be built simultaneously (all depend on ControlPlane API)
- Dashboard can start after Week 7 (API available)

### Operations
- Infrastructure setup in parallel (docker-compose, Proxmox, Terraform)
- CI/CD pipeline refinement
- Monitoring stack deployment

### Documentation
- Architecture decisions documented as they're made (ADRs)
- API docs auto-generated from OpenAPI spec
- User guides written during integration phase

---

## Sprint Planning (2-Week Cycles)

**Sprint 1 (Weeks 1-2):** Scaffolding, Shared Auth, Shared Monitoring  
**Sprint 2 (Weeks 3-4):** API Contracts, CI/CD, GitHub setup  
**Sprint 3 (Weeks 5-6):** ControlPlane foundation, DB schema  
**Sprint 4 (Weeks 7-8):** ControlPlane Core APIs + Monitoring  
**Sprint 5 (Weeks 9-10):** Gateway implementation  
**Sprint 6 (Weeks 11-12):** Mesh + DDNS foundation  
**Sprint 7 (Weeks 13-14):** Gateway ↔ ControlPlane integration  
**Sprint 8 (Weeks 15-16):** Mesh ↔ ControlPlane integration  
**Sprint 9 (Weeks 17-18):** DDNS Worker + Integration  
**Sprint 10 (Weeks 19-20):** Dashboard Frontend  
**Sprint 11 (Weeks 21-22):** Security Hardening  
**Sprint 12 (Weeks 23+):** Performance, Release Prep, v1.0  

---

## Success Metrics

| Milestone | Metric | Target |
|-----------|--------|--------|
| End Phase 1 | CI/CD pipelines functional | 100% |
| End Phase 2 | API test coverage | >80% |
| End Phase 2 | Shared library adoption | 5+ repos |
| End Phase 3 | End-to-end test pass rate | 100% |
| End Phase 3 | Dashboard feature complete | 100% |
| End Phase 4 | GitHub stars | 500+ |
| End Phase 4 | Docker pulls/month | 10k+ |
| End Phase 4 | Security audit findings | 0 critical |

---

**Last Updated:** June 5, 2026  
**Next Review:** Sprint planning kickoff (Week 1)
