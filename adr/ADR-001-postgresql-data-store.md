# ADR-001: PostgreSQL as Primary Data Store

**Date:** 2026-06-05  
**Status:** Accepted  
**Author:** @backend-lead  

## Context

The MeshGate ControlPlane requires a persistent data store for:
- Node/peer configuration
- User accounts and RBAC
- Job history and status
- Certificate management
- DNS records and status

We evaluated multiple options before selecting PostgreSQL.

## Decision

Use **PostgreSQL 14+** as the primary database for MeshGate ControlPlane.

## Rationale

### Why PostgreSQL?

| Requirement | PostgreSQL | MongoDB | DynamoDB | SQLite |
|------------|-----------|---------|----------|---------|
| ACID Transactions | ✅ Strong | ⚠️ Weak | ❌ No | ✅ Yes |
| JSON Support | ✅ JSONB | ✅ Native | ❌ No | ⚠️ Limited |
| Horizontal Scale | ⚠️ Via sharding | ✅ Built-in | ✅ Managed | ❌ No |
| Cost | ✅ Cheap | ✅ Cheap | ❌ Expensive | ✅ Free |
| Tooling | ✅ Excellent | ✅ Good | ⚠️ AWS-only | ⚠️ Limited |
| Query Power | ✅ Complex queries | ⚠️ Basic | ⚠️ Limited | ✅ Full SQL |
| Consistency | ✅ ACID | ⚠️ Eventual | ⚠️ Eventual | ✅ ACID |
| Self-hosted | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |

### Key Advantages

1. **Strong Consistency:** ACID transactions ensure configuration consistency across the system
   - Critical: Node configuration changes must be atomic
   - Job state must not have race conditions

2. **Rich Query Capabilities:** Complex queries for reporting and auditing
   - Find all nodes by location, status, or role
   - Audit trail queries with time ranges
   - Performance analysis queries

3. **JSON Support (JSONB):** Store flexible nested configs without sacrificing relational benefits
   ```sql
   SELECT * FROM nodes WHERE config->>'mesh_enabled' = 'true'
   ```

4. **Replication & HA:** Native streaming replication for high availability
   - Hot standby failover
   - Read replicas for reporting

5. **Cost:** Open-source, self-hosted, predictable costs
   - No vendor lock-in
   - Can run on any infrastructure (cloud, on-prem, edge)

6. **Mature Ecosystem:** Battle-tested libraries and tools
   - GORM in Go ecosystem
   - PgAdmin, pg_stat_statements for monitoring
   - Numerous backups tools (pg_dump, WAL-E, pgBackRest)

### Why NOT?

**MongoDB:** Weak consistency guarantees make configuration management risky. Job scheduling requires strong atomicity.

**DynamoDB:** Vendor lock-in, expensive, AWS-only. Conflicts with self-hosted philosophy.

**SQLite:** Single-file limitation, limited concurrency. Not suitable for multi-user system.

## Consequences

### Positive
- ✅ Transactional consistency for safety-critical operations
- ✅ Standard SQL allows complex ad-hoc queries
- ✅ JSONB for semi-structured config data
- ✅ Open-source, self-hosted, no vendor dependency
- ✅ Strong community and tooling
- ✅ Can scale to millions of nodes via sharding (future)

### Negative
- ❌ Requires explicit schema migrations (not zero-downtime auto-evolution)
- ❌ Sharding is more complex than MongoDB's auto-sharding
- ❌ Network latency for cross-datacenter deployments (vs. DynamoDB global tables)
- ❌ Requires operational expertise (backups, vacuum, monitoring)

### Mitigations

For schema evolution:
- Use migration tools (Flyway, Migrate)
- Test migrations in staging before production
- Plan for zero-downtime deployments (Phase 4)

For scaling:
- Horizontal read scaling via replicas
- Sharding strategy documented (Phase 4)
- Connection pooling via PgBouncer

## Alternatives Considered

### 1. MongoDB (Rejected)
```
Pros: Flexible schema, horizontal scaling
Cons: Eventual consistency, weak guarantees, overkill for this workload
```

### 2. DynamoDB (Rejected)
```
Pros: Managed service, global tables
Cons: Vendor lock-in, expensive, doesn't fit self-hosted model
```

### 3. SQLite (Rejected)
```
Pros: Zero configuration, embedded
Cons: Single-file limitation, no concurrency, no replication
```

### 4. CockroachDB (Considered, Deferred)
```
Pros: Distributed SQL, ACID at scale
Cons: Higher complexity, smaller community, more operational overhead
Status: Revisit for Phase 4 if global deployments needed
```

## Implementation Details

### Schema Design Principles
1. **Normalized by default:** Avoid data duplication
2. **Constraints enforced:** Use CHECK, UNIQUE, FOREIGN KEY constraints
3. **Audit trail:** Timestamps (created_at, updated_at) on all tables
4. **Soft deletes:** deleted_at column for reversible deletions (audit trail)

### Example Schema
```sql
CREATE TABLE nodes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  public_ip INET NOT NULL,
  mesh_ip INET NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',
  config JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMP,
  CONSTRAINT unique_name UNIQUE(name, deleted_at IS NULL),
  CONSTRAINT valid_status CHECK(status IN ('pending', 'active', 'inactive'))
);

CREATE INDEX idx_nodes_status ON nodes(status);
CREATE INDEX idx_nodes_created_at ON nodes(created_at DESC);
```

### Connection Management
- Use PgBouncer for connection pooling in production
- GORM with sensible pool settings:
  ```go
  MaxOpenConns: 25
  MaxIdleConns: 5
  ConnMaxLifetime: 5 * time.Minute
  ```

### Backup Strategy
- Daily automated backups via pgBackRest
- WAL archiving for point-in-time recovery
- Backup verification tests (Phase 4)

## Related ADRs

- [ADR-002: Redis for Job Queue](./ADR-002-redis-job-queue.md) (caching + sessions)
- [ADR-003: JWT for API Authentication](./ADR-003-jwt-authentication.md)

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [GORM Getting Started](https://gorm.io/docs/index.html)
- [PostgreSQL vs MongoDB Comparison](https://www.postgresql.org/about/press/presskit14/en/)

---

**Decision Record Version:** 1.0  
**Last Updated:** 2026-06-05
