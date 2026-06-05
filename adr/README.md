# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records for MeshGate. Each ADR documents a significant architectural decision, including context, rationale, and consequences.

## ADR Index

| # | Title | Status | Date | Impact |
|---|-------|--------|------|--------|
| [001](./ADR-001-postgresql-data-store.md) | PostgreSQL as Primary Data Store | Accepted | 2026-06-05 | Critical |
| [002](./ADR-002-redis-job-queue.md) | Redis for Job Queue & Caching | Pending | TBD | High |
| [003](./ADR-003-jwt-authentication.md) | JWT for API Authentication | Pending | TBD | Critical |
| [004](./ADR-004-wireguard-mesh.md) | WireGuard for Mesh Networking | Pending | TBD | Critical |
| [005](./ADR-005-go-backend-language.md) | Go as Primary Backend Language | Pending | TBD | Critical |

## ADR Process

### Creating a New ADR

1. **Proposal Phase:** Discuss in GitHub Discussions
2. **Draft:** Write ADR following template (see [ADR-001](./ADR-001-postgresql-data-store.md))
3. **Review:** Post PR, gather feedback from team leads
4. **Decision:** Project Lead approves/rejects
5. **Published:** Merge to main branch, status becomes "Accepted" or "Rejected"

### ADR Status Values

- **Accepted:** Decision has been approved and is in effect
- **Pending:** Decision awaiting approval from leadership
- **Rejected:** Decision was considered but not approved
- **Deprecated:** Previously accepted, but superseded by newer ADR
- **Proposed:** Early-stage idea under discussion

### ADR Template

```markdown
# ADR-NNN: Short Title

**Date:** YYYY-MM-DD  
**Status:** Proposed | Pending | Accepted | Rejected | Deprecated  
**Author:** @github-handle  

## Context
[Describe the situation requiring a decision]

## Decision
[What is the decision being made?]

## Rationale
[Why this decision? What are the tradeoffs?]

## Consequences
### Positive
- [Benefits]

### Negative
- [Drawbacks]

### Mitigations
- [How to address negative consequences]

## Alternatives Considered
- [Option 1: Why rejected?]
- [Option 2: Why rejected?]

## Related ADRs
- [ADR-XXX: Related Decision]

## References
- [Link to research/documentation]
```

## Decision Log

### Phase 1 (Foundation)
- [ ] ADR-001: PostgreSQL ✅ Accepted
- [ ] ADR-002: Redis for job queue
- [ ] ADR-003: JWT authentication
- [ ] ADR-004: WireGuard for mesh
- [ ] ADR-005: Go as backend language

### Phase 2 (Core Services)
- [ ] ADR-006: API versioning strategy
- [ ] ADR-007: RBAC model design
- [ ] ADR-008: Certificate management

### Phase 3 (Integration)
- [ ] ADR-009: Monitoring strategy
- [ ] ADR-010: Logging and tracing

### Phase 4 (Release)
- [ ] ADR-011: Scalability approach (sharding/replication)
- [ ] ADR-012: Deployment topology

---

**Last Updated:** 2026-06-05
