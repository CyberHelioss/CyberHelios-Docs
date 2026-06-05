# Local Development Setup

Complete guide for setting up MeshGate development environment on your machine.

---

## Prerequisites

### Required
- **Git** 2.40+
- **Go** 1.21+
- **Node.js** 18+ (LTS)
- **Docker** 24.0+
- **Docker Compose** 2.20+
- **Make** 4.0+

### Optional
- **PostgreSQL** 14+ (local client tools)
- **Redis** CLI tools
- **curl** or **Postman** (API testing)
- **VS Code** or **JetBrains IDE** (development)

### System Requirements
- **OS:** Linux, macOS, or Windows (WSL2)
- **CPU:** 2+ cores
- **RAM:** 4GB minimum, 8GB recommended
- **Disk:** 10GB free space

---

## Installation

### macOS (Homebrew)

```bash
# Install prerequisites
brew install go node docker docker-compose make git

# Install optional tools
brew install postgresql redis

# Verify installations
go version       # go version go1.21.0
node --version   # v18.x or v20.x
docker --version # Docker version 24.0+
```

### Ubuntu/Debian

```bash
# Update package manager
sudo apt update

# Install Go
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# Install Node.js (via NodeSource)
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Install other tools
sudo apt install -y make postgresql-client redis-tools git
```

### Windows (WSL2)

```bash
# In WSL2 terminal, follow Ubuntu/Debian steps above
wsl --install -d Ubuntu-22.04

# Then follow Ubuntu installation steps in WSL2 terminal
```

---

## Project Structure

```
CyberHelios/
├── CyberHelios-Docs/              # Documentation (this repo)
├── MeshGate-ControlPlain/         # Backend: Central orchestration
├── MeshGate-Gateway/              # Backend: Reverse proxy
├── MeshGate-Mesh/                 # Backend: Mesh networking
├── MeshGate-DDNS/                 # Backend: Dynamic DNS
├── MeshGate-DDNS-Worker/          # Backend: DDNS worker
├── MeshGate-Dashboard/            # Frontend: Web UI
├── CyberHelios-Shared-Auth/       # Shared: Authentication library
├── CyberHelios-Shared-Monitoring/ # Shared: Monitoring library
└── CyberHelios-infra/             # Infrastructure: Deployment code
```

---

## Quick Start (5 Minutes)

### 1. Clone Repositories

```bash
# Create workspace
mkdir ~/CyberHelios && cd ~/CyberHelios

# Clone all repositories
git clone https://github.com/CyberHelioss/CyberHelios-Docs.git
git clone https://github.com/CyberHelioss/MeshGate-ControlPlain.git
git clone https://github.com/CyberHelioss/MeshGate-Gateway.git
git clone https://github.com/CyberHelioss/MeshGate-Dashboard.git
git clone https://github.com/CyberHelioss/CyberHelios-Shared-Auth.git
git clone https://github.com/CyberHelioss/CyberHelios-Shared-Monitoring.git
```

### 2. Start Development Environment

```bash
# Navigate to ControlPlane (or any backend service)
cd MeshGate-ControlPlain

# Copy environment file
cp .env.example .env.local

# Start Docker containers
docker-compose up -d

# Verify services are running
docker-compose ps
```

### 3. Run Tests

```bash
# Backend
cd MeshGate-ControlPlain
make test
make lint

# Frontend
cd MeshGate-Dashboard
npm test
npm run lint
```

### 4. Start Services

```bash
# Terminal 1: Backend (ControlPlane)
cd MeshGate-ControlPlain
make run

# Terminal 2: Frontend (Dashboard)
cd MeshGate-Dashboard
npm run dev

# Services available at:
# - API: http://localhost:8080
# - Dashboard: http://localhost:3000
# - Swagger UI: http://localhost:8080/swagger/index.html
```

---

## Environment Configuration

### Backend Services (.env.local)

```bash
# Database
DATABASE_URL=postgres://user:password@localhost:5432/meshgate_dev
DATABASE_MIGRATIONS_PATH=./migrations

# Server
SERVER_PORT=8080
SERVER_ENV=development

# Authentication
JWT_SECRET=dev-secret-key-change-in-production
JWT_EXPIRY=15m
REFRESH_TOKEN_EXPIRY=7d

# Redis (optional)
REDIS_URL=redis://localhost:6379/0

# Logging
LOG_LEVEL=debug
LOG_FORMAT=json

# Monitoring
METRICS_PORT=9090
ENABLE_PPROF=true
```

### Frontend (.env.local)

```bash
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_API_VERSION=v1

# Auth
NEXT_PUBLIC_AUTH_REDIRECT=/dashboard

# Build
NODE_ENV=development
```

---

## Development Workflow

### Backend Development

#### Project Structure
```
MeshGate-ControlPlain/
├── main.go              # Entry point
├── cmd/                 # CLI commands
├── internal/
│   ├── api/             # HTTP handlers
│   ├── service/         # Business logic
│   ├── repository/      # Data access
│   ├── models/          # Domain models
│   └── config/          # Configuration
├── pkg/                 # Public packages
├── migrations/          # Database migrations
├── tests/               # Integration tests
├── Makefile             # Build automation
└── go.mod              # Dependencies
```

#### Common Commands
```bash
# Linting
make lint

# Testing
make test                 # Run all tests
make test-coverage       # Generate coverage report
make test-integration    # Integration tests with docker-compose
make test-race           # Race condition detection

# Building
make build               # Build binary
make run                 # Run locally

# Code generation
make generate            # Generate code (mocks, interfaces)

# Database
make migrate-up          # Run migrations
make migrate-down        # Revert migrations
```

### Frontend Development

#### Project Structure
```
MeshGate-Dashboard/
├── pages/               # Next.js pages
├── components/          # React components
├── hooks/               # Custom hooks
├── services/            # API clients
├── styles/              # Tailwind CSS
├── tests/               # Jest tests
├── public/              # Static assets
├── package.json
└── tsconfig.json
```

#### Common Commands
```bash
# Development
npm run dev              # Start dev server (http://localhost:3000)
npm test                 # Run tests
npm run lint             # ESLint checks
npm run lint:fix         # Auto-fix issues

# Building
npm run build            # Production build
npm start                # Run production build

# Code generation
npm run codegen          # Generate API clients from OpenAPI spec
```

---

## Database Management

### PostgreSQL Setup

```bash
# Start PostgreSQL in Docker
docker-compose up -d postgres

# Connect to database
psql postgresql://user:password@localhost:5432/meshgate_dev

# Or use pgAdmin (if configured)
# Access at http://localhost:5050
# Default: admin@admin.com / admin
```

### Database Migrations

```bash
# Create new migration
migrate create -ext sql -dir migrations -seq create_nodes_table

# Apply migrations
make migrate-up

# Rollback last migration
make migrate-down

# Check migration status
migrate -path migrations -database $DATABASE_URL version
```

### Seeding Development Data

```bash
# Run seed script
go run cmd/seed/main.go

# Or via Make
make seed
```

---

## Testing

### Backend Testing

```bash
# Unit tests only
go test ./...

# With coverage
go test -cover ./...

# Specific package
go test -v ./internal/api/handlers

# Run tests matching pattern
go test -run TestCreateNode ./...

# Race condition detection
go test -race ./...
```

### Frontend Testing

```bash
# Run all tests
npm test

# Watch mode (re-run on changes)
npm test -- --watch

# Coverage report
npm test -- --coverage

# Specific test file
npm test -- NodeForm.test.tsx
```

### Integration Testing

```bash
# Requires running services (docker-compose up)
make test-integration

# With specific focus
go test -run TestIntegration -integration ./...
```

---

## Debugging

### Backend Debugging

#### VS Code Debug Configuration
Create `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Connect to Go",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "program": "${fileDirname}",
      "env": {},
      "args": [],
      "showLog": true
    }
  ]
}
```

#### Command Line Debugging
```bash
# Start with delve debugger
dlv debug ./cmd/main.go

# Set breakpoint and debug
(dlv) break main.main
(dlv) continue
```

### Frontend Debugging

#### Browser DevTools
- Open http://localhost:3000 in Chrome/Firefox
- Press F12 or Cmd+Option+I
- Use React DevTools browser extension

#### VS Code Debugging
Create `.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "attach",
      "name": "Attach Chrome",
      "urlFilter": "http://localhost:3000/*",
      "port": 9222
    }
  ]
}
```

### Logs and Traces

```bash
# View container logs
docker-compose logs -f controlplane

# View specific service logs
docker-compose logs -f postgres

# Tail logs with live update
docker-compose logs -f --tail=100
```

---

## Troubleshooting

### Port Already in Use
```bash
# Find process using port 8080
lsof -i :8080

# Kill process
kill -9 <PID>

# Or use different port
PORT=8081 make run
```

### Database Connection Issues
```bash
# Check if PostgreSQL is running
docker-compose ps postgres

# Restart PostgreSQL
docker-compose restart postgres

# Reset database (WARNING: deletes all data)
docker-compose down -v
docker-compose up -d postgres
make migrate-up
```

### Module Import Errors
```bash
# Clear Go mod cache
go clean -modcache

# Re-download dependencies
go mod download

# Tidy dependencies
go mod tidy
```

### Node Modules Issues
```bash
# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

---

## Performance Profiling

### Go CPU Profiling
```bash
# Start with CPU profiling
go run -cpuprofile=cpu.prof main.go

# Analyze profile
go tool pprof cpu.prof
```

### Go Memory Profiling
```bash
# Generate memory profile
curl http://localhost:6060/debug/pprof/heap > mem.prof

# Analyze profile
go tool pprof mem.prof
```

### Frontend Performance
```bash
# Analyze bundle size
npm run build -- --analyze

# Run Lighthouse
npm install -g @lhci/cli@latest
lhci autorun
```

---

## IDE Setup

### VS Code Extensions

Recommended extensions for `.vscode/extensions.json`:
```json
{
  "recommendations": [
    "golang.go",
    "ms-vscode.makefile-tools",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "ms-vscode-remote.remote-wsl",
    "GitHub.copilot"
  ]
}
```

### GoLand / IntelliJ IDEA
- Enable code inspections: Settings → Editor → Inspections
- Configure Go SDK: Settings → Go → Go Modules
- Set up test runner: Settings → Languages & Frameworks → Go

---

## Git Workflow

### Branch Naming
```
feature/<name>          # New features
fix/<name>              # Bug fixes
docs/<name>             # Documentation
refactor/<name>         # Refactoring
perf/<name>             # Performance improvements
```

### Commit Messages
```
feat(auth): add JWT token refresh endpoint

- Implemented refresh token validation
- Added 7-day expiry for refresh tokens
- Added integration tests

Closes #123
```

### Pre-Commit Hooks

Create `.git/hooks/pre-commit`:
```bash
#!/bin/bash
# Lint and test before commit
make lint && make test
```

---

## CI/CD Local Testing

### Run GitHub Actions Locally (Act)

```bash
# Install act
brew install act

# Run all workflows
act

# Run specific workflow
act -j build

# Run with specific event
act pull_request
```

---

## Additional Resources

- [Go Documentation](https://golang.org/doc/)
- [React Documentation](https://react.dev/)
- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MeshGate Architecture](./ARCHITECTURE.md)
- [Contributing Guidelines](./CONTRIBUTING.md)

---

## Getting Help

- **GitHub Issues:** Report bugs and feature requests
- **GitHub Discussions:** Ask questions and brainstorm
- **Email:** dev@cyberhelios.dev
- **Documentation:** See README in each repo

---

**Last Updated:** June 5, 2026  
**Next Update:** Phase 1 completion
