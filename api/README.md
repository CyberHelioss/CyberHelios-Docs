# API Specifications

This directory contains OpenAPI/gRPC specifications for all MeshGate APIs.

## Available APIs

### Control Plane REST API
**File:** `control-plane-openapi.yaml`  
**Status:** Draft (Phase 1)  
**Purpose:** Central orchestration API for managing nodes, peers, configuration, and jobs

**Base URL:** `https://controlplane.example.com/api/v1`

**Key Endpoints:**
- Authentication: `POST /auth/login`, `POST /auth/refresh`
- Nodes: `GET/POST/PUT/DELETE /nodes`
- Peers: `GET /peers`, `POST /peers/{id}/discover`
- Jobs: `GET/POST/PATCH /jobs`
- Configuration: `GET/PATCH /config`
- Certificates: `GET/POST /certs`
- DNS: `GET/POST /dns/records`

### Gateway Admin API (Optional)
**File:** `gateway-admin-openapi.yaml`  
**Status:** TBD (Phase 2)  
**Purpose:** Manage gateway routing rules and certificates

### Mesh Node API (Internal)
**File:** `mesh-node-openapi.yaml`  
**Status:** TBD (Phase 2)  
**Purpose:** Node-to-node and node-to-controlplane communication

---

## Code Generation

### Generate Go Client
```bash
# From control-plane-openapi.yaml
openapi-generator generate -i control-plane-openapi.yaml -g go -o generated/go-client
```

### Generate TypeScript Client (for Dashboard)
```bash
# From control-plane-openapi.yaml
openapi-generator generate -i control-plane-openapi.yaml -g typescript-fetch -o generated/ts-client
```

### Generate Mock Server (for Testing)
```bash
# Using Prism
prism mock control-plane-openapi.yaml -p 8888
```

---

## Adding New APIs

1. **Design Phase:** Discuss in GitHub Discussions
2. **Specification:** Write OpenAPI 3.0 YAML
3. **Validation:** Run `npx @openapitools/openapi-generator-cli validate -i <file>`
4. **Review:** Post PR for team review
5. **Generation:** Generate code and integrate into services
6. **Documentation:** Add endpoint docs and examples

---

**Last Updated:** 2026-06-05
