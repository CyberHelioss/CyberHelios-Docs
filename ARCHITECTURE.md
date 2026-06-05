# MeshGate Architecture

## High-Level Overview

MeshGate is a distributed system with clear separation of concerns. Traffic flows through the Gateway, services communicate via APIs, and infrastructure is managed through the Control Plane.

```
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET / ISP                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  MeshGate       │
                    │  Gateway        │
                    │  (Reverse Proxy)│
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
  ┌─────▼────────┐  ┌──────▼──────┐  ┌──────────▼──┐
  │  Internal    │  │  Control    │  │  Mesh       │
  │  Services    │  │  Plane API  │  │  Network    │
  │              │  │  (REST)     │  │  (Tunnels)  │
  └──────────────┘  └──────┬──────┘  └─────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼──────┐  ┌──────▼─────┐  ┌─────────▼────┐
   │PostgreSQL │  │ Redis      │  │ Dashboard UI │
   │ (config)  │  │ (jobs)     │  │ (React)      │
   └───────────┘  └────────────┘  └──────────────┘

        ↓ DDNS Updates DNS Records ↓
   
   ┌──────────────────────────────────┐
   │  DNS Provider (Route53, CF, etc) │
   └──────────────────────────────────┘
```

## System Components

### 1. MeshGate Gateway
**Role:** Public-facing reverse proxy  
**Responsibilities:**
- HTTPS termination with automatic cert renewal
- Hostname-based routing to internal services
- Rate limiting and DDoS protection
- Security headers (HSTS, CSP, etc.)
- Optional admin endpoints
- Request/response logging

**Dependencies:** None (edge service)  
**Exposes:** HTTPS port 443 to internet  
**Technology:** Caddy or Nginx + Go middleware  

### 2. MeshGate Control Plane
**Role:** Orchestration and policy engine  
**Responsibilities:**
- Node/peer registration and management
- Configuration storage and distribution
- Certificate lifecycle management
- Policy and ACL enforcement
- Route definitions
- Job scheduling (DDNS, health checks)
- Event log and audit trails

**Dependencies:** PostgreSQL, Redis (optional)  
**Exposes:** REST API + WebSocket for real-time updates  
**Technology:** Go + Echo/Gin framework  

### 3. MeshGate Mesh
**Role:** Encrypted private networking layer  
**Responsibilities:**
- Peer discovery and enrollment
- Encrypted tunnel creation (WireGuard or custom)
- NAT traversal (hole punching, relay)
- Route optimization
- ACL enforcement at network layer
- Health monitoring

**Dependencies:** Control Plane API  
**Deployed on:** Each node (client agent)  
**Technology:** Go + WireGuard  

### 4. MeshGate DDNS Service
**Role:** Dynamic DNS record management  
**Responsibilities:**
- Multi-provider DNS support (Route53, Cloudflare, etc.)
- IPv4/IPv6 tracking
- Retry logic with exponential backoff
- Change detection and atomic updates
- Status reporting to Control Plane

**Dependencies:** Control Plane API  
**Technology:** Go + DNS client libraries  

### 5. MeshGate DDNS Worker
**Role:** Distributed job executor  
**Responsibilities:**
- Polls Control Plane for jobs
- Executes DDNS updates
- Monitors public IP changes
- Reports health and status
- Retries failed jobs

**Dependencies:** Control Plane API, DDNS service  
**Deployed on:** One or more nodes  
**Technology:** Go  

### 6. MeshGate Dashboard
**Role:** Web administration interface  
**Responsibilities:**
- Node/peer visualization
- Configuration management UI
- Real-time monitoring (metrics, logs)
- Certificate management
- DNS status and history
- User authentication and RBAC

**Dependencies:** Control Plane API  
**Technology:** React + Next.js + Tailwind CSS  

### 7. CyberHelios Shared Auth
**Role:** Authentication and authorization library  
**Responsibilities:**
- JWT issuance and validation
- RBAC (Role-Based Access Control)
- Session management
- Token refresh logic
- Auth middleware for all services

**Consumed by:** All APIs (Gateway, Control Plane, Dashboard)  
**Technology:** Go + standard JWT libraries  

### 8. CyberHelios Shared Monitoring
**Role:** Observability infrastructure  
**Responsibilities:**
- Metrics collection (Prometheus format)
- Structured logging (JSON to Loki)
- Health check endpoints
- Distributed tracing (OpenTelemetry optional)

**Consumed by:** All services  
**Technology:** Go + Prometheus client + structured logging  

### 9. CyberHelios Infrastructure
**Role:** Deployment and operations automation  
**Responsibilities:**
- Infrastructure provisioning (Terraform)
- Configuration management (Ansible)
- Container orchestration (Docker Compose, Kubernetes optional)
- Backup and disaster recovery
- Monitoring stack deployment
- CI/CD pipelines

**Deployed on:** Proxmox, LXC, Docker, bare metal  
**Technology:** Terraform + Ansible + Docker  

## Data Flow

### User Accessing Internal Service
1. User resolves `service.example.com` → points to MeshGate public IP (via DDNS)
2. Browser connects to Gateway on HTTPS
3. Gateway checks TLS cert (auto-renewed by Control Plane)
4. Gateway routes to internal service based on hostname rules
5. Service response flows back through Gateway to user

### Node Joining the Mesh
1. Node agent registers with Control Plane
2. Control Plane validates credentials via Auth library
3. Node receives peer list and ACL rules
4. Mesh layer establishes encrypted tunnels to peers
5. Health checks run periodically
6. Mesh monitors for route changes

### IP Change Detected
1. DDNS Worker detects new public IP
2. Reports to Control Plane
3. Control Plane triggers DDNS update job
4. DDNS service updates DNS records at provider
5. Dashboard shows status update
6. External clients see new IP within TTL

## API Contracts

### Control Plane API
All APIs use REST with JSON payload. Authentication via JWT bearer token.

**Key Endpoints:**
- `POST /api/v1/nodes` — Register a new node
- `GET /api/v1/nodes` — List all nodes
- `PUT /api/v1/nodes/{id}` — Update node config
- `GET /api/v1/peers` — List mesh peers
- `POST /api/v1/jobs` — Create job (DDNS, etc)
- `GET /api/v1/config` — Get system configuration
- `POST /api/v1/auth/login` — Authenticate user

See `/api/control-plane-openapi.yaml` for full specification.

## Dependency Graph

```
CyberHelios-Shared-Auth
├─ MeshGate-Gateway
├─ MeshGate-ControlPlane
├─ MeshGate-Dashboard
└─ MeshGate-Mesh

CyberHelios-Shared-Monitoring
├─ MeshGate-ControlPlane
├─ MeshGate-Gateway
├─ MeshGate-Mesh
└─ MeshGate-DDNS

MeshGate-ControlPlane
├─ MeshGate-Gateway
├─ MeshGate-Mesh
├─ MeshGate-DDNS-Worker
├─ MeshGate-Dashboard
└─ CyberHelios-infra

PostgreSQL ← MeshGate-ControlPlane
Redis ← MeshGate-ControlPlane (optional)
```

**Build Order (for parallel development):**
1. Start: CyberHelios-Shared-Auth, CyberHelios-Shared-Monitoring
2. Then: MeshGate-ControlPlane (depends on shared libs)
3. Parallel: Gateway, Mesh, DDNS (all depend on ControlPlane API)
4. Finally: Dashboard (consumes ControlPlane APIs)

## Security Model

### Authentication
- All APIs require JWT bearer tokens
- Tokens issued by Control Plane `/auth/login` endpoint
- Tokens signed with HS256 or RS256
- Short TTL (15 minutes), refresh tokens for longer sessions

### Authorization
- RBAC with roles: `admin`, `operator`, `read-only`
- Roles stored in JWT claims
- Control Plane enforces role checks on API endpoints
- Dashboard UI hides/disables unauthorized actions

### Encryption
- All inter-service communication uses HTTPS/TLS
- Mesh networking uses WireGuard (end-to-end encryption)
- Database credentials stored in environment (12-factor app)
- Secrets never logged

### Network Isolation
- Gateway is public-facing only
- All internal services private (behind mesh or firewall)
- Mesh enforces peer-to-peer ACLs
- Control Plane accessible only from internal network

## Deployment Topology

### Development
```
Docker Compose on single machine
- PostgreSQL container
- Redis container
- ControlPlane service
- Gateway service
- Dashboard service
- Mock DDNS provider
```

### Production
```
Proxmox + LXC Containers
- Control Plane VM (HA setup with 3 nodes)
- Gateway VM (load balanced)
- Database cluster (PostgreSQL HA)
- Redis cluster (optional)
- Monitoring stack (Prometheus, Grafana, Loki)
- Multiple mesh nodes for resilience
```

## Performance Considerations

- **Latency:** Sub-100ms for API calls (within datacenter)
- **Throughput:** Gateway handles 10k+ req/sec per instance (tuned)
- **Scalability:** Stateless services scale horizontally
- **Database:** Connection pooling via PgBouncer
- **Caching:** Redis for job queue and config caching

---

**Last Updated:** June 5, 2026
