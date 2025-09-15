# Local CI/CD Setup Guide

This guide covers how to run the complete CI/CD pipeline locally, including Docker setup, testing workflows, and Cypress E2E tests.

## Overview

The DealSphere platform uses a comprehensive CI/CD pipeline that includes:

- **Backend Tests** - Spring Boot unit and integration tests
- **Frontend Tests** - React/TypeScript unit tests and linting
- **E2E Tests** - Cypress browser automation testing
- **Code Quality** - SonarCloud analysis and security scanning
- **Docker Integration** - Full stack deployment testing

## Prerequisites

Before running the CI/CD setup locally, ensure you have:

### Required Software

```bash
# Package Managers
✅ Node.js 20+
✅ pnpm 8+
✅ Java 17+
✅ Docker & Docker Compose

# Development Tools
✅ Git
✅ curl (for health checks)
```

### Installation Commands

```bash
# Node.js & pnpm
curl -fsSL https://get.pnpm.io/install.sh | sh
pnpm env use --global 20

# Java 17 (using SDKMAN)
curl -s "https://get.sdkman.io" | bash
sdk install java 17.0.7-tem

# Docker
# Install Docker Desktop from: https://docker.com/products/docker-desktop
```

## Quick Start

### 1. Clone and Setup

```bash
# Clone the repository
git clone <repository-url>
cd dealsphere-platform

# Copy environment configuration
cp .env.development .env

# Verify Docker is running
docker --version
docker-compose --version
```

### 2. Run Full CI/CD Pipeline Locally

```bash
# Start the complete application stack
docker-compose up -d --build

# Wait for services to be ready (this mimics CI behavior)
./scripts/wait-for-services.sh

# Run all tests in sequence (like CI)
./scripts/run-local-ci.sh
```

## Detailed CI/CD Components

### Backend Testing

The backend uses Spring Boot with comprehensive test coverage:

```bash
cd backend

# Run unit tests
./gradlew test

# Run with coverage
./gradlew test jacocoTestReport

# Run integration tests
./gradlew integrationTest

# Test with real database
./gradlew test -Dspring.profiles.active=test
```

**Test Configuration:**
- **Unit Tests**: Fast, mocked dependencies
- **Integration Tests**: Real database, Docker Testcontainers
- **Test Data**: DataInitializer with service classes only

### Frontend Testing

The frontend uses Vitest and Cypress for comprehensive testing:

```bash
cd frontend

# Install dependencies
pnpm install

# Run unit tests
pnpm test

# Run with coverage
pnpm test:coverage

# Linting and type checking
pnpm lint
pnpm type-check

# Build verification
pnpm build
```

### E2E Testing with Cypress

Cypress tests run against the full application stack:

#### Local Development Testing

```bash
# Ensure application is running
docker-compose up -d

# Wait for services (backend health check)
curl -f http://localhost/actuator/health

# Run Cypress tests
cd frontend
pnpm cy:run

# Or run interactively
pnpm cy:open
```

#### Test Configuration

The E2E tests verify:
- ✅ **Login Flow** - Complete authentication workflow
- ✅ **JWT Storage** - Token persistence in localStorage
- ✅ **Redux Saga** - State management integration
- ✅ **Navigation** - Dashboard redirects and routing
- ✅ **Error Handling** - Network failures and invalid credentials

**Test Files:**
```
cypress/
├── e2e/
│   └── auth/
│       ├── login-flow.cy.ts      # Complete login workflow
│       ├── jwt-storage.cy.ts     # Token storage verification
│       └── basic-login.cy.ts     # Basic functionality
├── support/
│   ├── commands.ts               # Custom Cypress commands
│   └── e2e.ts                   # Global configuration
└── fixtures/
    └── users.json               # Test user data
```

## Environment Configuration

### Environment Files

The project uses environment-specific configuration:

```bash
.env.development    # Local development (http://localhost)
.env.staging       # Staging environment
.env.production    # Production environment
```

**Key Variables:**
```bash
# Backend Configuration
SPRING_PROFILES_ACTIVE=docker
POSTGRES_DB=dealsphere_dev
JWT_SECRET=<secure-key>

# Frontend Configuration
VITE_API_URL=http://localhost/api
VITE_GRAPHQL_URL=http://localhost/api/graphql

# Cypress Testing
CYPRESS_BASE_URL=http://localhost
CYPRESS_VIDEO=true
CYPRESS_SCREENSHOTS=true
```

### Docker Configuration

```yaml
# docker-compose.yml structure
services:
  db:          # PostgreSQL 16 database
  backend:     # Spring Boot application
  frontend:    # React/Vite development server
  nginx:       # Reverse proxy (port 80)
```

## Local CI/CD Workflows

### 1. Full Pipeline Simulation

Create this script to simulate the complete CI/CD pipeline:

```bash
#!/bin/bash
# scripts/run-local-ci.sh

set -e

echo "🚀 Starting Local CI/CD Pipeline"

# 1. Backend Tests
echo "📊 Running Backend Tests..."
cd backend
./gradlew clean test
cd ..

# 2. Frontend Tests
echo "⚛️ Running Frontend Tests..."
cd frontend
pnpm install
pnpm lint
pnpm test
pnpm build
cd ..

# 3. Docker Build Test
echo "🐳 Testing Docker Build..."
docker-compose build --no-cache
docker-compose up -d

# 4. Wait for Services
echo "⏳ Waiting for services to be ready..."
timeout 120 bash -c 'until curl -f http://localhost/actuator/health > /dev/null 2>&1; do sleep 5; done'

# 5. E2E Tests
echo "🧪 Running E2E Tests..."
cd frontend
pnpm cy:run
cd ..

# 6. Cleanup
echo "🧹 Cleaning up..."
docker-compose down -v

echo "✅ Local CI/CD Pipeline Complete!"
```

### 2. Service Health Checks

Create a health check script:

```bash
#!/bin/bash
# scripts/wait-for-services.sh

set -e

echo "⏳ Waiting for services to be ready..."

# Check PostgreSQL
echo "🗄️ Checking PostgreSQL..."
timeout 60 bash -c 'until docker-compose exec db pg_isready -U dealsphere_dev > /dev/null 2>&1; do sleep 2; done'

# Check Backend
echo "🌐 Checking Backend..."
timeout 120 bash -c 'until curl -f http://localhost/actuator/health > /dev/null 2>&1; do sleep 5; done'

# Check GraphQL
echo "📡 Checking GraphQL..."
timeout 60 bash -c 'until curl -f http://localhost/api/graphql -H "Content-Type: application/json" -d "{\"query\":\"query{__typename}\"}" > /dev/null 2>&1; do sleep 2; done'

# Check Frontend
echo "⚛️ Checking Frontend..."
timeout 60 bash -c 'until curl -f http://localhost > /dev/null 2>&1; do sleep 2; done'

echo "✅ All services are ready!"
```

## Troubleshooting

### Common Issues

#### 1. Port Conflicts
```bash
# Check if ports are in use
lsof -i :80    # Nginx
lsof -i :5432  # PostgreSQL
lsof -i :5173  # Vite dev server

# Kill processes if needed
sudo lsof -ti:80 | xargs kill -9
```

#### 2. Docker Issues
```bash
# Clean Docker state
docker-compose down -v
docker system prune -f
docker volume prune -f

# Rebuild from scratch
docker-compose build --no-cache
```

#### 3. Database Connection Issues
```bash
# Check PostgreSQL logs
docker-compose logs db

# Connect to database manually
docker-compose exec db psql -U dealsphere_dev -d dealsphere_dev

# Reset database
docker-compose down -v
docker-compose up -d db
```

#### 4. Cypress Test Failures
```bash
# Run with debug output
cd frontend
DEBUG=cypress:* pnpm cy:run

# Check video recordings
open cypress/videos/

# Run specific test
pnpm cy:run --spec "cypress/e2e/auth/login-flow.cy.ts"
```

### Performance Optimization

#### Speed Up Local Development

```bash
# Use Docker layer caching
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1

# Pre-pull images
docker-compose pull

# Use pnpm store for faster installs
pnpm config set store-dir ~/.pnpm-store
```

#### Parallel Testing

```bash
# Run tests in parallel (requires multiple environments)
docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d

# Run Cypress tests in parallel
pnpm cy:run --record --parallel --key <cypress-record-key>
```

## Integration with IDE

### VS Code Setup

**.vscode/settings.json:**
```json
{
  "docker.host": "unix:///var/run/docker.sock",
  "typescript.preferences.importModuleSpecifier": "relative",
  "jest.jestCommandLine": "pnpm test",
  "cypress.showActionsInCommandPalette": true
}
```

**.vscode/tasks.json:**
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Start Local CI/CD",
      "type": "shell",
      "command": "./scripts/run-local-ci.sh",
      "group": "test",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "new"
      }
    }
  ]
}
```

## Next Steps

1. **Set up GitHub Actions** - See [GitHub Actions Guide](github-actions.md)
2. **Configure SonarCloud** - See [Code Quality Guide](code-quality.md)
3. **Deploy to Staging** - See [Deployment Guide](deployment.md)
4. **Monitor Production** - See [Monitoring Guide](monitoring.md)

## Related Documentation

- [GitHub Actions Workflows](github-actions.md)
- [Cypress E2E Testing](cypress-e2e-testing.md)
- [Docker Development](docker-development.md)
- [Environment Configuration](environment-config.md)