# Contributing to MeshGate

Thank you for interest in contributing to MeshGate! This guide covers code standards, PR process, testing requirements, and community guidelines.

---

## Code of Conduct

We are committed to providing a welcoming and inspiring community for all. Read our [Code of Conduct](./CODE_OF_CONDUCT.md) before contributing.

---

## Getting Started

### Prerequisites

- Go 1.21+ (for backend services)
- Node.js 18+ (for Dashboard)
- Docker & Docker Compose
- Git

### Local Development Setup

```bash
# Clone the main repos
git clone https://github.com/CyberHelioss/CyberHelios-Docs.git
git clone https://github.com/CyberHelioss/MeshGate-ControlPlain.git
git clone https://github.com/CyberHelioss/MeshGate-Gateway.git
git clone https://github.com/CyberHelioss/MeshGate-Dashboard.git

# Set up local environment
cd CyberHelios-Docs
cp .env.example .env.local
docker-compose up -d

# Verify environment
make check-env  # Backend
npm run check   # Frontend
```

---

## Code Style Guide

### Go Backend

#### Naming Conventions
```go
// Good
func (s *Service) CreateNode(ctx context.Context, req *CreateNodeRequest) (*Node, error)
type NodeRepository interface
var nodeCount int

// Bad
func (s *Service) create_node(req CreateNodeRequest) Node
type node_repository interface
var NC int
```

#### Formatting
- Use `gofmt` for automatic formatting
- Line length: 120 characters max
- Indent with tabs (Go standard)

#### Best Practices
```go
// Error handling
if err != nil {
  return nil, fmt.Errorf("failed to create node: %w", err)
}

// Logging (structured JSON)
logger.Info("Node created", "node_id", node.ID, "name", node.Name)

// Comments on exported functions
// CreateNode creates a new node in the system and registers it with the mesh network.
func (s *Service) CreateNode(ctx context.Context, req *CreateNodeRequest) (*Node, error)

// Testing
func TestCreateNode_Success(t *testing.T)
func TestCreateNode_InvalidInput(t *testing.T)
```

#### Imports Organization
```go
import (
  // Standard library
  "context"
  "fmt"
  
  // Third-party
  "github.com/gin-gonic/gin"
  
  // Internal
  "github.com/CyberHelioss/MeshGate-ControlPlain/internal/models"
)
```

### React/TypeScript Frontend

#### Naming Conventions
```tsx
// Good
function NodeManagementPanel() { }
const useNodeData = () => { }
interface NodeFormProps { }

// Bad
function node_management_panel() { }
const use_node_data = () => { }
interface nodeFormProps { }
```

#### Component Structure
```tsx
// Functional component with clear exports
export interface NodeDetailsProps {
  nodeId: string;
  onUpdate?: (node: Node) => void;
}

export function NodeDetails({ nodeId, onUpdate }: NodeDetailsProps) {
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState<Error | null>(null);

  // Logic here
  
  return (
    <div className="...">
      {/* JSX */}
    </div>
  );
}
```

#### ESLint & Prettier
```bash
# Check
npm run lint

# Fix
npm run lint:fix

# Format
npm run format
```

---

## Testing Requirements

### Backend (Go)

#### Minimum Coverage
- Core logic: 80%+
- API handlers: 70%+
- Database layer: 75%+

#### Testing Patterns
```go
func TestCreateNode_Success(t *testing.T) {
  // Arrange
  repo := NewMockNodeRepository()
  service := NewNodeService(repo)
  req := &CreateNodeRequest{
    Name: "test-node",
    IP:   "192.168.1.10",
  }
  
  // Act
  node, err := service.CreateNode(context.Background(), req)
  
  // Assert
  require.NoError(t, err)
  require.NotNil(t, node)
  assert.Equal(t, "test-node", node.Name)
}

func TestCreateNode_InvalidIP(t *testing.T) {
  // Test error cases
}
```

#### Running Tests
```bash
make test                    # Run all tests
make test-coverage          # Generate coverage report
make test-race              # Run with race detector
make test-integration       # Integration tests with docker-compose
```

### Frontend (React)

#### Minimum Coverage
- Components: 70%+
- Hooks: 75%+
- Utils: 80%+

#### Testing Patterns
```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { NodeForm } from './NodeForm';

describe('NodeForm', () => {
  it('submits form with valid data', async () => {
    const handleSubmit = jest.fn();
    render(<NodeForm onSubmit={handleSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/name/i), {
      target: { value: 'test-node' }
    });
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    expect(handleSubmit).toHaveBeenCalled();
  });
});
```

#### Running Tests
```bash
npm test                    # Run all tests
npm run test:coverage       # Coverage report
npm run test:watch         # Watch mode
```

---

## PR Process

### Before Submitting

1. **Create a branch from `main`:**
   ```bash
   git checkout -b feature/node-creation
   # or
   git checkout -b fix/auth-token-leak
   ```

2. **Make changes** following code style guide

3. **Run tests locally:**
   ```bash
   make lint test           # Backend
   npm run lint && npm test # Frontend
   ```

4. **Write clear commit messages:**
   ```
   fix: resolve auth token expiration issue
   
   - Token now refreshes 5 minutes before expiry
   - Added integration test for refresh flow
   - Closes #123
   ```

### PR Template

```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issues
Closes #123

## Testing
- [ ] Added unit tests
- [ ] Added integration tests
- [ ] Manual testing completed

## Security Checklist
- [ ] No hardcoded secrets
- [ ] Input validation added
- [ ] Output encoding applied
- [ ] Auth checks in place

## Screenshots (if applicable)
<!-- For UI changes -->
```

### PR Review Checklist

Reviewers will check:

- ✅ Code follows style guide
- ✅ Tests added/updated
- ✅ Coverage maintained (>80% for new code)
- ✅ Documentation updated
- ✅ No breaking changes without discussion
- ✅ Security checklist completed
- ✅ Commits are logical and well-messaged

### Merging

- PRs require 2 approvals (core team) or 1 approval (community)
- CI/CD pipeline must pass
- Branches must be up-to-date with main
- Squash commits before merge (default)

---

## Documentation Standards

### Code Comments
```go
// Package models contains domain models for MeshGate.
package models

// Node represents a single machine in the MeshGate network.
type Node struct {
  // ID is the unique identifier for the node.
  ID string
  
  // Name is the human-readable name of the node.
  Name string `json:"name" validate:"required,max=255"`
}
```

### README Structure
Each repo should have a README with:
1. **Overview** — What this package does in 1-2 sentences
2. **Quick Start** — Get running in 5 minutes
3. **API Docs** — Link to full specification
4. **Contributing** — Link to this guide
5. **License** — GPL-3.0

### ADRs (Architecture Decision Records)

Use for significant architectural decisions:

```markdown
# ADR-001: Use PostgreSQL for Control Plane Storage

**Date:** 2026-06-05  
**Status:** Accepted

## Decision
Use PostgreSQL for the ControlPlane database rather than MongoDB.

## Rationale
- Strong ACID guarantees needed for configuration consistency
- JSON support for flexible schemas
- Better tooling and monitoring support

## Consequences
- +: ACID transactions
- +: Better performance for relational queries
- -: Less flexible for rapid schema changes
- -: Requires migration planning

## Alternatives Considered
- MongoDB (rejected: weak consistency)
- DynamoDB (rejected: cost, vendor lock-in)
```

---

## Reporting Issues

### Bug Reports
Include:
- **OS/Environment:** Linux 6.1, Docker 24.0
- **Reproduction steps:** Exact steps to reproduce
- **Expected behavior:** What should happen
- **Actual behavior:** What actually happens
- **Logs/stack trace:** Full error output

### Feature Requests
Include:
- **Use case:** Why is this needed?
- **Proposed solution:** How should it work?
- **Alternatives:** Other approaches considered

---

## Commit Message Standards

Format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `refactor:` Code refactoring
- `perf:` Performance improvement
- `test:` Test addition/modification
- `chore:` Build, CI, dependencies

Example:
```
feat(mesh): add NAT traversal via UPnP

- Detect external IP via UPnP IGD
- Automatically forward ports
- Fallback to relay if UPnP unavailable

Closes #456
```

---

## Code Review Expectations

### For Reviewers
- ✅ Review within 24 hours if marked urgent
- ✅ Be constructive and respectful
- ✅ Explain the reasoning behind feedback
- ✅ Approve when satisfied, don't block on minor issues

### For Authors
- ✅ Respond to feedback promptly
- ✅ Ask clarifying questions if feedback unclear
- ✅ Respect reviewer time, don't over-iterate

---

## Release Process

### Version Numbering
Semantic versioning: `MAJOR.MINOR.PATCH`

- `MAJOR:` Breaking changes
- `MINOR:` New features (backward compatible)
- `PATCH:` Bug fixes

### Cutting a Release
```bash
# 1. Create release branch
git checkout -b release/v1.0.0

# 2. Update version in relevant files
# 3. Update CHANGELOG.md
# 4. Create PR, get reviews
# 5. Merge to main
git tag v1.0.0
git push origin v1.0.0

# GitHub Actions automatically:
# - Runs full test suite
# - Builds docker images
# - Pushes to Docker Hub
# - Creates GitHub release
```

---

## Communication Channels

- **GitHub Issues:** Bug reports, feature requests
- **GitHub Discussions:** Design proposals, questions
- **GitHub PRs:** Code reviews, feedback
- **Email:** security@cyberhelios.dev (security issues only)

---

## License

By contributing, you agree that your contributions will be licensed under the [GPL-3.0 license](./LICENSE).

---

**Last Updated:** June 5, 2026  
**Maintained By:** @AmRam841 and core team
