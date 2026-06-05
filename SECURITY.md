# MeshGate Security Model

## Overview

Security is a first-class concern in MeshGate. This document defines the authentication, authorization, encryption, and threat model for the system.

---

## Authentication

### JWT-Based Authentication

All API requests must include a bearer token:

```
Authorization: Bearer <JWT_TOKEN>
```

#### Token Structure

```json
{
  "sub": "user-id",
  "email": "user@example.com",
  "role": "admin",
  "org": "org-id",
  "exp": 1234567890,
  "iat": 1234567890,
  "iss": "meshgate-controlplane"
}
```

#### Token Lifecycle

- **Access Token:** Valid for 15 minutes
- **Refresh Token:** Valid for 7 days
- **Token Signing:** HS256 (symmetric) or RS256 (public key)

#### Endpoints

```
POST /api/v1/auth/login
  Request: { email: string, password: string }
  Response: { accessToken: string, refreshToken: string, expiresIn: number }

POST /api/v1/auth/refresh
  Request: { refreshToken: string }
  Response: { accessToken: string, expiresIn: number }

POST /api/v1/auth/logout
  Request: (empty)
  Response: { status: "ok" }
```

---

## Authorization (RBAC)

### Role Model

Three roles are defined:

| Role | Permissions | Use Case |
|------|-------------|----------|
| **admin** | Full system access, user management | System administrator |
| **operator** | Node/peer management, DDNS updates, job triggers | Team member |
| **read-only** | View all resources, no modifications | Audit/monitoring |

### Role Hierarchy

- `admin` ⊃ `operator` ⊃ `read-only`
- Admin can grant/revoke roles
- Permissions are additive (higher role includes lower)

### API Endpoint Protection

```go
// Pseudo-code for middleware
func requireRole(role string) Middleware {
  return func(c Context) {
    token := extractToken(c)
    claims := parseJWT(token)
    
    if !hasRole(claims, role) {
      c.Error(403, "Forbidden")
      return
    }
    c.Next()
  }
}

// Usage:
router.PUT("/nodes/:id", requireRole("operator"), updateNode)
router.GET("/audit-logs", requireRole("admin"), getAuditLogs)
```

### Resource-Level ACLs (Future)

In Phase 4, implement per-resource ACLs:
- User A can manage Mesh Nodes in org A
- User B can only view dashboards
- Service account has limited token scope

---

## Encryption

### Transport Layer

**All inter-service communication uses HTTPS/TLS 1.3 minimum.**

#### Certificate Management

- **Internal Services:** Self-signed certs (dev) or internal CA (prod)
- **Public Gateway:** ACME certificates (Let's Encrypt)
- **Auto-Renewal:** ControlPlane manages certificate lifecycle
- **Expiration Buffer:** Renew 30 days before expiry

#### TLS Configuration

```go
// Minimum TLS configuration
tlsConfig := &tls.Config{
  MinVersion:               tls.VersionTLS13,
  PreferServerCipherSuites: true,
  CipherSuites: []uint16{
    tls.TLS_AES_256_GCM_SHA384,
    tls.TLS_CHACHA20_POLY1305_SHA256,
  },
}
```

### Application Layer

#### Mesh Network Encryption

Mesh networking uses **WireGuard** for peer-to-peer encryption:
- End-to-end encryption between nodes
- No plaintext traffic on untrusted networks
- Perfect forward secrecy

#### Data-at-Rest

- Database credentials stored as environment variables (12-factor app)
- Sensitive config stored encrypted in database (AES-256-GCM)
- Private keys stored with file permissions 0600
- Backups encrypted with master key

---

## Secrets Management

### Secret Types

| Secret | Storage | Rotation | Access |
|--------|---------|----------|--------|
| Database password | Environment variable | On deployment | ControlPlane only |
| JWT signing key | Environment variable | Annually or on breach | All services |
| API keys (external) | Encrypted DB | Per provider | DDNS service |
| SSH keys | Encrypted DB | Per key policy | Infrastructure only |
| TLS certificates | Filesystem / ACME | Auto | Gateway, ControlPlane |

### Secret Distribution

1. **Development:** `.env.local` files (never committed)
2. **Staging:** Secrets manager (AWS Secrets Manager, HashiCorp Vault)
3. **Production:** Secrets manager with audit logging

### Secret Rotation

- JWT keys: Rotate annually or after suspected breach
- API keys: Rotate quarterly
- Passwords: Rotate on hiring/departure
- SSH keys: Rotate on key compromise

---

## Network Security

### Network Topology

```
UNTRUSTED (Internet)
    ↓
[Gateway - Public IP]
    ↓
SEMI-TRUSTED (Internal Network)
    ↓
[ControlPlane] ↔ [Mesh Agents] ↔ [Internal Services]
    ↓
TRUSTED (Private Database, Redis)
```

### Firewall Rules

```
# Gateway rules (public)
ALLOW 0.0.0.0/0 → :443/tcp (HTTPS)
ALLOW 0.0.0.0/0 → :80/tcp (HTTP → HTTPS redirect)
DENY all other ports

# Internal rules
ALLOW internal → ControlPlane:8080/tcp (REST API)
ALLOW internal → ControlPlane:8081/tcp (WebSocket)
ALLOW internal → PostgreSQL:5432/tcp (DB access from ControlPlane only)
ALLOW internal ↔ internal:51820/udp (Mesh tunnels)
```

### DDoS Protection

- Rate limiting: 1000 req/minute per IP on public APIs
- Request size limits: 10MB max body
- Connection timeouts: 30s idle timeout
- Request timeouts: 60s per request
- Anti-bot headers: Check User-Agent, accept only known tools

---

## Input Validation

### Rule: Validate Everything

All inputs are treated as untrusted:

```go
// Example validation
type NodeRegistration struct {
  Name    string `json:"name" validate:"required,max=255,alphanumspace"`
  PublicIP string `json:"public_ip" validate:"required,ip"`
  MeshIP  string `json:"mesh_ip" validate:"required,ip"`
}

// Middleware
func validateInput(c Context) {
  var req NodeRegistration
  if err := c.BindJSON(&req); err != nil {
    c.Error(400, "Invalid JSON")
    return
  }
  if err := validate.Struct(req); err != nil {
    c.Error(400, "Validation failed: " + err.Error())
    return
  }
  c.Next()
}
```

### Output Encoding

- **HTML responses:** HTML escape all user input
- **JSON responses:** JSON encode special characters
- **Database:** Use parameterized queries (NEVER string concatenation)

### Database Security

```go
// GOOD - Parameterized query
node, err := db.Query("SELECT * FROM nodes WHERE id = ?", nodeID)

// BAD - SQL Injection vulnerability
node, err := db.Query("SELECT * FROM nodes WHERE id = " + nodeID)
```

---

## Audit Logging

### What to Log

- Login/logout events
- API calls (endpoint, user, timestamp, IP, status code)
- Configuration changes (what changed, who, when)
- Permission changes (who granted/revoked)
- Errors and security events

### Log Format

All logs in JSON format for structured logging:

```json
{
  "timestamp": "2026-06-05T10:00:00Z",
  "level": "INFO",
  "service": "controlplane",
  "event": "api_call",
  "method": "PUT",
  "path": "/api/v1/nodes/123",
  "user_id": "user-456",
  "status": 200,
  "duration_ms": 45,
  "client_ip": "192.168.1.100",
  "message": "Node updated successfully"
}
```

### Log Storage

- **Development:** Console (stdout)
- **Production:** Loki (centralized log storage)
- **Retention:** 90 days default, 1 year for security events

---

## Threat Model

### Assets

| Asset | Value | Impact if Compromised |
|-------|-------|----------------------|
| User credentials | High | Account takeover |
| JWT signing key | Critical | Token forgery (full system access) |
| TLS private keys | Critical | MITM attacks, data interception |
| Database | High | All user data, configs leaked |
| Mesh network | High | Internal services exposed |
| API credentials (external) | Medium | Unauthorized DNS updates |

### Threats

| Threat | Probability | Impact | Mitigation |
|--------|------------|--------|-----------|
| Brute force login | Medium | Account takeover | Rate limiting, account lockout |
| SQL injection | Low | DB compromise | Parameterized queries, input validation |
| MITM on public API | Medium | Data interception | HTTPS/TLS enforcement |
| MITM on internal API | Low | Internal data leak | Network segmentation, encryption |
| Token forgery | Low | Full system access | Strong signing key, short TTL |
| Privilege escalation | Low | Unauthorized changes | RBAC enforcement, audit logs |
| Code injection (XSS) | Medium | Session hijacking | Output encoding, CSP headers |
| Denial of Service | Medium | Service unavailable | Rate limiting, DDoS protection |

### Attack Scenarios & Mitigations

#### Scenario 1: Attacker brute-forces login
- **Mitigation:**
  - Rate limit: 5 attempts per minute per IP
  - Account lockout: 10 failed attempts = 15 minute lockout
  - Log suspicious activity
  - Alert on repeated failures

#### Scenario 2: Attacker intercepts API traffic
- **Mitigation:**
  - Enforce HTTPS/TLS 1.3 globally
  - HSTS header with 1-year max-age
  - Certificate pinning (optional, Phase 4)
  - Network monitoring for unusual patterns

#### Scenario 3: Insider obtains JWT signing key
- **Mitigation:**
  - Rotate signing keys annually
  - Audit who accessed key management systems
  - Revoke all existing tokens on key compromise
  - Short token TTL (15 minutes)

#### Scenario 4: Database is compromised
- **Mitigation:**
  - Encrypt sensitive fields (AES-256)
  - Use strong password hashing (bcrypt)
  - Database backups encrypted
  - Principle of least privilege (ControlPlane account has minimal permissions)

---

## Security Checklist for PRs

All PRs must address:

- [ ] No hardcoded secrets or credentials
- [ ] All user inputs validated and sanitized
- [ ] All outputs properly encoded
- [ ] Auth required on sensitive endpoints
- [ ] Appropriate role/permission checks
- [ ] Database queries use parameterized statements
- [ ] HTTPS/TLS enforced where needed
- [ ] Error messages don't leak sensitive info
- [ ] Logging includes audit trail
- [ ] New dependencies reviewed for vulnerabilities
- [ ] Tests include security-specific cases

---

## Incident Response

### In Case of Security Breach

1. **Immediate (0-1 hour):**
   - Isolate compromised systems
   - Revoke all active JWT tokens
   - Alert security team
   - Begin forensics

2. **Short-term (1-24 hours):**
   - Identify root cause
   - Patch vulnerability
   - Rotate all secrets/keys
   - Deploy hotfix
   - Notify affected users

3. **Medium-term (24-72 hours):**
   - Post-mortem analysis
   - Root cause mitigation
   - Security audit of related code
   - Update security model if needed

4. **Long-term:**
   - Bug bounty program
   - Penetration testing
   - Security training for team

---

## Compliance

### GDPR (if EU users)
- [ ] Consent for data collection
- [ ] Data export functionality
- [ ] Right to be forgotten (data deletion)
- [ ] Privacy policy
- [ ] Data processing agreements with vendors

### HIPAA (if health data)
- [ ] Encryption at rest and in transit
- [ ] Access logs and audit trails
- [ ] Business associate agreements
- [ ] Risk assessments

---

## Future Enhancements (Phase 4+)

- [ ] Two-factor authentication (2FA)
- [ ] API key-based authentication for service accounts
- [ ] OAuth2 / OpenID Connect integration
- [ ] Single sign-on (SSO) via SAML
- [ ] Per-resource ACLs
- [ ] End-to-end encryption for sensitive data
- [ ] Hardware security module (HSM) for key storage
- [ ] Intrusion detection system (IDS)
- [ ] Security information and event management (SIEM)

---

**Last Updated:** June 5, 2026  
**Next Review:** Phase 2 kickoff (Week 5)
