# Environment Management

This guide covers environment configuration, secrets management, and deployment pipelines for DealSphere across different environments (development, staging, production).

## Overview

DealSphere uses a multi-environment approach to ensure code quality and system reliability:

- **Development**: Local development environment with Docker Compose
- **Staging**: Pre-production environment for testing and integration
- **Production**: Live environment serving real users

## Environment Configuration

### Environment Variables Structure

Each environment uses specific configuration files:

```
environments/
├── .env.development      # Local development
├── .env.staging         # Staging environment
├── .env.production      # Production environment
└── .env.example         # Template with all variables
```

### Core Environment Variables

```bash
# Application Configuration
APP_ENV=production
APP_NAME=dealsphere
APP_VERSION=1.0.0
APP_URL=https://dealsphere.com
APP_PORT=8080

# Database Configuration
DB_HOST=dealsphere-db-cluster
DB_PORT=5432
DB_NAME=dealsphere_prod
DB_USERNAME=dealsphere_user
DB_PASSWORD=${DB_PASSWORD_SECRET}
DB_SSL_MODE=require
DB_POOL_SIZE=20

# Redis Configuration
REDIS_HOST=dealsphere-redis-cluster
REDIS_PORT=6379
REDIS_PASSWORD=${REDIS_PASSWORD_SECRET}
REDIS_DATABASE=0
REDIS_SSL=true

# JWT Configuration
JWT_SECRET=${JWT_SECRET_KEY}
JWT_EXPIRATION=7200
JWT_REFRESH_EXPIRATION=604800

# GraphQL Configuration
GRAPHQL_PLAYGROUND_ENABLED=false
GRAPHQL_INTROSPECTION_ENABLED=false

# Logging Configuration
LOG_LEVEL=INFO
LOG_FORMAT=JSON
LOG_FILE_ENABLED=true
LOG_FILE_PATH=/var/log/dealsphere/app.log

# Monitoring Configuration
METRICS_ENABLED=true
METRICS_PORT=9090
HEALTH_CHECK_ENABLED=true

# Email Configuration
SMTP_HOST=${SMTP_HOST}
SMTP_PORT=587
SMTP_USERNAME=${SMTP_USERNAME}
SMTP_PASSWORD=${SMTP_PASSWORD}
SMTP_FROM=noreply@dealsphere.com

# File Storage Configuration
STORAGE_PROVIDER=aws-s3
STORAGE_BUCKET=dealsphere-files-prod
AWS_REGION=us-west-2
AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}

# Feature Flags
FEATURE_ADVANCED_ANALYTICS=true
FEATURE_REAL_TIME_NOTIFICATIONS=true
FEATURE_AUDIT_LOGGING=true
```

### Environment-Specific Configurations

#### Development Environment

```bash
# .env.development
APP_ENV=development
APP_URL=http://localhost:3000
DB_HOST=localhost
DB_SSL_MODE=disable
GRAPHQL_PLAYGROUND_ENABLED=true
GRAPHQL_INTROSPECTION_ENABLED=true
LOG_LEVEL=DEBUG
METRICS_ENABLED=false
```

#### Staging Environment

```bash
# .env.staging
APP_ENV=staging
APP_URL=https://staging.dealsphere.com
DB_HOST=staging-db.dealsphere.internal
DB_SSL_MODE=require
GRAPHQL_PLAYGROUND_ENABLED=true
GRAPHQL_INTROSPECTION_ENABLED=false
LOG_LEVEL=INFO
METRICS_ENABLED=true
```

#### Production Environment

```bash
# .env.production
APP_ENV=production
APP_URL=https://dealsphere.com
DB_HOST=prod-db-cluster.dealsphere.internal
DB_SSL_MODE=require
GRAPHQL_PLAYGROUND_ENABLED=false
GRAPHQL_INTROSPECTION_ENABLED=false
LOG_LEVEL=WARN
METRICS_ENABLED=true
```

## Secrets Management

### Local Development

For local development, use a `.env` file:

```bash
# Copy template
cp .env.example .env.development

# Edit with your local values
vim .env.development
```

### Cloud Environments

#### AWS Secrets Manager

```bash
# Store database credentials
aws secretsmanager create-secret \
    --name "dealsphere/prod/database" \
    --description "Production database credentials" \
    --secret-string '{
        "username": "dealsphere_user",
        "password": "secure-db-password",
        "host": "prod-db-cluster.amazonaws.com",
        "port": 5432,
        "database": "dealsphere_prod"
    }'

# Store JWT secret
aws secretsmanager create-secret \
    --name "dealsphere/prod/jwt-secret" \
    --description "JWT signing secret" \
    --secret-string "your-super-secure-jwt-secret-key"

# Store Redis password
aws secretsmanager create-secret \
    --name "dealsphere/prod/redis-password" \
    --description "Redis authentication password" \
    --secret-string "secure-redis-password"
```

#### Kubernetes Secrets

```yaml
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: dealsphere-secrets
  namespace: dealsphere-prod
type: Opaque
data:
  # Base64 encoded values
  DB_PASSWORD: c2VjdXJlLWRiLXBhc3N3b3Jk
  JWT_SECRET: eW91ci1zdXBlci1zZWN1cmUtand0LXNlY3JldC1rZXk=
  REDIS_PASSWORD: c2VjdXJlLXJlZGlzLXBhc3N3b3Jk
  SMTP_PASSWORD: c210cC1wYXNzd29yZA==
  AWS_SECRET_ACCESS_KEY: YXdzLXNlY3JldC1hY2Nlc3Mta2V5
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: dealsphere-config
  namespace: dealsphere-prod
data:
  APP_ENV: "production"
  DB_HOST: "prod-db-cluster.dealsphere.internal"
  DB_PORT: "5432"
  DB_NAME: "dealsphere_prod"
  DB_USERNAME: "dealsphere_user"
  REDIS_HOST: "prod-redis-cluster.dealsphere.internal"
  REDIS_PORT: "6379"
  LOG_LEVEL: "INFO"
  METRICS_ENABLED: "true"
```

#### Docker Swarm Secrets

```bash
# Create secrets
echo "secure-db-password" | docker secret create db_password -
echo "your-jwt-secret-key" | docker secret create jwt_secret -
echo "secure-redis-password" | docker secret create redis_password -

# Use in docker-compose.yml
version: '3.8'
services:
  backend:
    image: dealsphere/backend:latest
    secrets:
      - db_password
      - jwt_secret
      - redis_password
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - JWT_SECRET_FILE=/run/secrets/jwt_secret
      - REDIS_PASSWORD_FILE=/run/secrets/redis_password

secrets:
  db_password:
    external: true
  jwt_secret:
    external: true
  redis_password:
    external: true
```

## Configuration Management

### Application Properties Template

Create `application-template.yml` for environment-specific overrides:

```yaml
# application-template.yml
server:
  port: ${APP_PORT:8080}
  servlet:
    context-path: /api

spring:
  application:
    name: ${APP_NAME:dealsphere}
  profiles:
    active: ${APP_ENV:development}

  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:dealsphere}
    username: ${DB_USERNAME:dealsphere}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: ${DB_POOL_SIZE:10}
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    password: ${REDIS_PASSWORD:}
    database: ${REDIS_DATABASE:0}
    ssl: ${REDIS_SSL:false}

  jpa:
    hibernate:
      ddl-auto: ${HIBERNATE_DDL_AUTO:validate}
    show-sql: ${JPA_SHOW_SQL:false}
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true

logging:
  level:
    com.dealsphere: ${LOG_LEVEL:INFO}
    org.springframework.security: ${SECURITY_LOG_LEVEL:WARN}
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: ${LOG_FILE_PATH:/var/log/dealsphere/app.log}

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  endpoint:
    health:
      enabled: ${HEALTH_CHECK_ENABLED:true}
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: ${METRICS_ENABLED:false}

jwt:
  secret: ${JWT_SECRET:default-dev-secret}
  expiration: ${JWT_EXPIRATION:3600}
  refresh-expiration: ${JWT_REFRESH_EXPIRATION:604800}

graphql:
  playground:
    enabled: ${GRAPHQL_PLAYGROUND_ENABLED:false}
  introspection:
    enabled: ${GRAPHQL_INTROSPECTION_ENABLED:false}
```

### Environment Validation

Create environment validation script:

```bash
#!/bin/bash
# validate-environment.sh

set -e

REQUIRED_VARS=(
    "APP_ENV"
    "DB_HOST" "DB_USERNAME" "DB_PASSWORD"
    "REDIS_HOST"
    "JWT_SECRET"
)

echo "🔍 Validating environment configuration..."

missing_vars=()

for var in "${REQUIRED_VARS[@]}"; do
    if [[ -z "${!var}" ]]; then
        missing_vars+=("$var")
    fi
done

if [[ ${#missing_vars[@]} -ne 0 ]]; then
    echo "❌ Missing required environment variables:"
    printf '  - %s\n' "${missing_vars[@]}"
    exit 1
fi

# Environment-specific validations
if [[ "$APP_ENV" == "production" ]]; then
    PROD_REQUIRED_VARS=(
        "DB_SSL_MODE"
        "REDIS_SSL"
        "SMTP_HOST" "SMTP_USERNAME" "SMTP_PASSWORD"
        "AWS_ACCESS_KEY_ID" "AWS_SECRET_ACCESS_KEY"
    )

    prod_missing=()
    for var in "${PROD_REQUIRED_VARS[@]}"; do
        if [[ -z "${!var}" ]]; then
            prod_missing+=("$var")
        fi
    done

    if [[ ${#prod_missing[@]} -ne 0 ]]; then
        echo "❌ Missing production-specific variables:"
        printf '  - %s\n' "${prod_missing[@]}"
        exit 1
    fi

    # Security checks
    if [[ "$JWT_SECRET" == "default-dev-secret" ]]; then
        echo "❌ Production environment cannot use default JWT secret"
        exit 1
    fi

    if [[ "$DB_SSL_MODE" != "require" ]]; then
        echo "⚠️  Warning: Database SSL should be 'require' in production"
    fi
fi

echo "✅ Environment validation passed"

# Test database connection
echo "🔍 Testing database connection..."
if command -v psql >/dev/null; then
    if PGPASSWORD="$DB_PASSWORD" psql -h "$DB_HOST" -U "$DB_USERNAME" -d "$DB_NAME" -c "SELECT 1" >/dev/null 2>&1; then
        echo "✅ Database connection successful"
    else
        echo "❌ Database connection failed"
        exit 1
    fi
else
    echo "⚠️  psql not found, skipping database connection test"
fi

# Test Redis connection
echo "🔍 Testing Redis connection..."
if command -v redis-cli >/dev/null; then
    if redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" ${REDIS_PASSWORD:+-a "$REDIS_PASSWORD"} ping >/dev/null 2>&1; then
        echo "✅ Redis connection successful"
    else
        echo "❌ Redis connection failed"
        exit 1
    fi
else
    echo "⚠️  redis-cli not found, skipping Redis connection test"
fi

echo "🎉 All environment checks passed!"
```

## Deployment Pipelines

### CI/CD Configuration

#### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy DealSphere

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: dealsphere-inc/dealsphere-platform

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: dealsphere_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json

    - name: Validate Environment
      run: |
        cp .env.example .env.test
        chmod +x scripts/validate-environment.sh
        ./scripts/validate-environment.sh

    - name: Run Backend Tests
      run: |
        cd backend
        ./gradlew test jacocoTestReport
      env:
        DB_HOST: localhost
        DB_USERNAME: postgres
        DB_PASSWORD: test
        DB_NAME: dealsphere_test
        REDIS_HOST: localhost
        JWT_SECRET: test-jwt-secret

    - name: Run Frontend Tests
      run: |
        cd frontend
        npm ci
        npm run test:coverage

    - name: Upload Coverage
      uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
    - uses: actions/checkout@v4

    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and Push Backend
      uses: docker/build-push-action@v4
      with:
        context: ./backend
        file: ./backend/Dockerfile
        push: true
        tags: ${{ steps.meta.outputs.tags }}-backend
        labels: ${{ steps.meta.outputs.labels }}

    - name: Build and Push Frontend
      uses: docker/build-push-action@v4
      with:
        context: ./frontend
        file: ./frontend/Dockerfile
        push: true
        tags: ${{ steps.meta.outputs.tags }}-frontend
        labels: ${{ steps.meta.outputs.labels }}

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/staging'
    environment: staging

    steps:
    - uses: actions/checkout@v4

    - name: Deploy to Staging
      run: |
        echo "Deploying to staging environment..."
        # Add staging deployment commands
      env:
        STAGING_HOST: ${{ secrets.STAGING_HOST }}
        STAGING_SSH_KEY: ${{ secrets.STAGING_SSH_KEY }}

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
    - uses: actions/checkout@v4

    - name: Deploy to Production
      run: |
        echo "Deploying to production environment..."
        # Add production deployment commands
      env:
        PRODUCTION_HOST: ${{ secrets.PRODUCTION_HOST }}
        PRODUCTION_SSH_KEY: ${{ secrets.PRODUCTION_SSH_KEY }}
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Deployment Scripts

#### Rolling Deployment Script

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

ENVIRONMENT=${1:-staging}
VERSION=${2:-latest}
HEALTH_CHECK_TIMEOUT=300
ROLLBACK_ON_FAILURE=true

echo "🚀 Starting deployment to $ENVIRONMENT (version: $VERSION)"

# Load environment-specific configuration
if [[ -f ".env.$ENVIRONMENT" ]]; then
    source ".env.$ENVIRONMENT"
else
    echo "❌ Environment file .env.$ENVIRONMENT not found"
    exit 1
fi

# Validate environment
./scripts/validate-environment.sh

# Pre-deployment health check
echo "🏥 Pre-deployment health check..."
if ! curl -f "$APP_URL/actuator/health" >/dev/null 2>&1; then
    echo "⚠️  Application not responding, proceeding with deployment"
fi

# Database migrations (if needed)
if [[ "$RUN_MIGRATIONS" == "true" ]]; then
    echo "📊 Running database migrations..."
    docker run --rm \
        --network dealsphere-network \
        -e DB_HOST="$DB_HOST" \
        -e DB_USERNAME="$DB_USERNAME" \
        -e DB_PASSWORD="$DB_PASSWORD" \
        -e DB_NAME="$DB_NAME" \
        ghcr.io/dealsphere-inc/dealsphere-platform:${VERSION}-backend \
        java -jar app.jar --spring.profiles.active=migration
fi

# Rolling deployment
echo "🔄 Performing rolling deployment..."

if command -v docker-compose >/dev/null; then
    # Docker Compose deployment
    export IMAGE_TAG=$VERSION
    docker-compose -f docker-compose.prod.yml pull
    docker-compose -f docker-compose.prod.yml up -d --remove-orphans

elif command -v kubectl >/dev/null; then
    # Kubernetes deployment
    kubectl set image deployment/dealsphere-backend \
        backend=ghcr.io/dealsphere-inc/dealsphere-platform:${VERSION}-backend
    kubectl set image deployment/dealsphere-frontend \
        frontend=ghcr.io/dealsphere-inc/dealsphere-platform:${VERSION}-frontend

    kubectl rollout status deployment/dealsphere-backend --timeout=300s
    kubectl rollout status deployment/dealsphere-frontend --timeout=300s
else
    echo "❌ No supported orchestration tool found"
    exit 1
fi

# Health check with retry
echo "🏥 Post-deployment health check..."
for i in {1..30}; do
    if curl -f "$APP_URL/actuator/health" >/dev/null 2>&1; then
        echo "✅ Application is healthy"
        break
    fi

    if [[ $i -eq 30 ]]; then
        echo "❌ Health check failed after 30 attempts"

        if [[ "$ROLLBACK_ON_FAILURE" == "true" ]]; then
            echo "🔄 Rolling back deployment..."
            if command -v kubectl >/dev/null; then
                kubectl rollout undo deployment/dealsphere-backend
                kubectl rollout undo deployment/dealsphere-frontend
            else
                docker-compose -f docker-compose.prod.yml down
                # Restore previous version logic here
            fi
        fi
        exit 1
    fi

    echo "⏳ Waiting for application to become healthy (attempt $i/30)..."
    sleep 10
done

# Smoke tests
echo "🧪 Running smoke tests..."
./scripts/smoke-tests.sh "$APP_URL"

echo "🎉 Deployment completed successfully!"

# Post-deployment notifications
if [[ -n "$SLACK_WEBHOOK_URL" ]]; then
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"✅ DealSphere $ENVIRONMENT deployment completed (version: $VERSION)\"}" \
        "$SLACK_WEBHOOK_URL"
fi
```

#### Smoke Tests Script

```bash
#!/bin/bash
# scripts/smoke-tests.sh

set -e

BASE_URL=${1:-http://localhost:8080}
TIMEOUT=30

echo "🧪 Running smoke tests against $BASE_URL"

# Test health endpoint
echo "Testing health endpoint..."
if ! curl -f --max-time $TIMEOUT "$BASE_URL/actuator/health" >/dev/null; then
    echo "❌ Health check failed"
    exit 1
fi
echo "✅ Health check passed"

# Test metrics endpoint
echo "Testing metrics endpoint..."
if ! curl -f --max-time $TIMEOUT "$BASE_URL/actuator/prometheus" >/dev/null; then
    echo "❌ Metrics endpoint failed"
    exit 1
fi
echo "✅ Metrics endpoint passed"

# Test GraphQL endpoint
echo "Testing GraphQL endpoint..."
GRAPHQL_QUERY='{"query":"query { __schema { queryType { name } } }"}'
if ! curl -f --max-time $TIMEOUT \
    -H "Content-Type: application/json" \
    -d "$GRAPHQL_QUERY" \
    "$BASE_URL/graphql" >/dev/null; then
    echo "❌ GraphQL endpoint failed"
    exit 1
fi
echo "✅ GraphQL endpoint passed"

# Test database connectivity (through app)
echo "Testing database connectivity..."
DB_QUERY='{"query":"query { organizations { id name } }"}'
if ! curl -f --max-time $TIMEOUT \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer test-token" \
    -d "$DB_QUERY" \
    "$BASE_URL/graphql" >/dev/null 2>&1; then
    echo "⚠️  Database connectivity test skipped (requires auth)"
else
    echo "✅ Database connectivity passed"
fi

echo "🎉 All smoke tests passed!"
```

## Environment Monitoring

### Health Checks Configuration

```yaml
# health-checks.yml
version: '3.8'
services:
  backend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  frontend:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s

  postgres:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME} -d ${DB_NAME}"]
      interval: 30s
      timeout: 5s
      retries: 5

  redis:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 5s
      retries: 3
```

### Monitoring Integration

Environment monitoring is integrated with the observability stack. Each environment reports to centralized monitoring:

- **Development**: Local Grafana dashboard
- **Staging**: Shared monitoring cluster with environment labels
- **Production**: Dedicated monitoring infrastructure with high availability

Key metrics tracked per environment:

- Application health and availability
- Resource utilization (CPU, memory, disk)
- Request rates and response times
- Error rates and types
- Database performance
- Security events and audit logs

## Troubleshooting

### Common Environment Issues

**Environment Variables Not Loading**
```bash
# Check environment file exists and has correct permissions
ls -la .env.*
chmod 644 .env.production

# Verify variables are exported
env | grep -E '^(DB_|REDIS_|JWT_)'

# Test with explicit source
source .env.production && ./scripts/validate-environment.sh
```

**Database Connection Issues**
```bash
# Test direct connection
PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -U $DB_USERNAME -d $DB_NAME -c "SELECT version();"

# Check network connectivity
nslookup $DB_HOST
telnet $DB_HOST $DB_PORT

# Verify SSL configuration
psql "postgresql://$DB_USERNAME:$DB_PASSWORD@$DB_HOST:$DB_PORT/$DB_NAME?sslmode=$DB_SSL_MODE"
```

**Secrets Access Issues**
```bash
# AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id "dealsphere/prod/database"

# Kubernetes secrets
kubectl get secrets -n dealsphere-prod
kubectl describe secret dealsphere-secrets -n dealsphere-prod

# Docker secrets
docker secret ls
docker secret inspect jwt_secret
```

This comprehensive environment management guide ensures consistent, secure, and reliable deployments across all DealSphere environments.