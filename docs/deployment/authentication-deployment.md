# Authentication System Deployment Guide

## Overview

This guide provides comprehensive deployment instructions for the DealSphere authentication system, covering production deployment, security configuration, and operational procedures.

## Prerequisites

### System Requirements

- **Java Runtime**: OpenJDK 17+
- **Node.js**: v18+ with pnpm
- **Database**: PostgreSQL 14+
- **Web Server**: nginx 1.20+
- **Container Runtime**: Docker 20+ or Kubernetes 1.24+

### External Services

- **SMTP Service**: For sending invitation and password reset emails
- **Observability Stack**: For security event logging (ELK, Datadog, etc.)
- **SSL Certificates**: Valid TLS certificates for HTTPS

## Environment Configuration

### Production Environment Variables

Create a `.env.production` file with the following variables:

```bash
# Application Configuration
NODE_ENV=production
SERVER_PORT=8080
CLIENT_URL=https://app.dealsphere.com

# Database Configuration
DATABASE_URL=postgresql://username:password@db-host:5432/dealsphere_prod
DATABASE_SSL=true
DATABASE_POOL_SIZE=20
DATABASE_CONNECTION_TIMEOUT=30000

# JWT Configuration
JWT_SECRET=your-super-secure-256-bit-secret-key-here
JWT_EXPIRATION_MS=3600000  # 1 hour for production
JWT_REFRESH_EXPIRATION_MS=604800000  # 7 days

# Email Configuration
SMTP_HOST=smtp.mailgun.com
SMTP_PORT=587
SMTP_USERNAME=postmaster@mg.dealsphere.com
SMTP_PASSWORD=secure-smtp-password
SMTP_FROM_ADDRESS=noreply@dealsphere.com
SMTP_FROM_NAME="DealSphere Platform"
SMTP_USE_TLS=true

# Security Configuration
BCRYPT_ROUNDS=12
RATE_LIMIT_ENABLED=true
RATE_LIMIT_LOGIN_MAX=5
RATE_LIMIT_LOGIN_WINDOW_MS=900000  # 15 minutes
RATE_LIMIT_RESET_MAX=3
RATE_LIMIT_RESET_WINDOW_MS=3600000  # 1 hour

# Password Policy
PASSWORD_MIN_LENGTH=8
PASSWORD_RESET_EXPIRY_MINUTES=15
INVITATION_EXPIRY_HOURS=48

# CORS Configuration
CORS_ALLOWED_ORIGINS=https://app.dealsphere.com,https://admin.dealsphere.com
CORS_ALLOWED_METHODS=GET,POST,PUT,DELETE,OPTIONS
CORS_ALLOWED_HEADERS=Origin,Content-Type,Accept,Authorization
CORS_ALLOW_CREDENTIALS=true

# External Observability
OBSERVABILITY_ENDPOINT=https://api.observability-provider.com
OBSERVABILITY_API_KEY=your-observability-api-key
LOG_LEVEL=INFO
STRUCTURED_LOGGING=true

# Feature Flags
FEATURE_INVITATION_SYSTEM=true
FEATURE_PASSWORD_RESET=true
FEATURE_SECURITY_AUDITING=true
FEATURE_RATE_LIMITING=true
```

### Docker Compose Production

**File**: `docker-compose.prod.yml`

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx/nginx.prod.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - backend
      - frontend
    restart: unless-stopped
    networks:
      - dealsphere-network

  backend:
    image: dealsphere/backend:${VERSION:-latest}
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DATABASE_URL=${DATABASE_URL}
      - JWT_SECRET=${JWT_SECRET}
      - SMTP_HOST=${SMTP_HOST}
      - SMTP_USERNAME=${SMTP_USERNAME}
      - SMTP_PASSWORD=${SMTP_PASSWORD}
    volumes:
      - ./logs/backend:/app/logs
    secrets:
      - jwt_secret
      - db_password
      - smtp_password
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    networks:
      - dealsphere-network
    depends_on:
      - database

  frontend:
    image: dealsphere/frontend:${VERSION:-latest}
    environment:
      - NODE_ENV=production
      - VITE_API_URL=https://api.dealsphere.com
      - VITE_GRAPHQL_URL=https://api.dealsphere.com/graphql
    volumes:
      - ./logs/frontend:/app/logs
    restart: unless-stopped
    networks:
      - dealsphere-network

  database:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=dealsphere_prod
      - POSTGRES_USER=dealsphere
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/backups:/backups
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dealsphere -d dealsphere_prod"]
      interval: 30s
      timeout: 10s
      retries: 5
    restart: unless-stopped
    networks:
      - dealsphere-network

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - dealsphere-network

volumes:
  postgres_data:
  redis_data:

secrets:
  jwt_secret:
    external: true
  db_password:
    external: true
  smtp_password:
    external: true

networks:
  dealsphere-network:
    driver: bridge
```

### Kubernetes Deployment

**File**: `k8s/auth-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dealsphere-backend
  labels:
    app: dealsphere-backend
    component: authentication
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: dealsphere-backend
  template:
    metadata:
      labels:
        app: dealsphere-backend
        component: authentication
    spec:
      containers:
      - name: backend
        image: dealsphere/backend:latest
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: jwt-secret
              key: secret
        - name: SMTP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: smtp-secret
              key: password
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: logs
          mountPath: /app/logs
      volumes:
      - name: tmp
        emptyDir: {}
      - name: logs
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: dealsphere-backend-service
spec:
  selector:
    app: dealsphere-backend
  ports:
  - name: http
    port: 8080
    targetPort: 8080
  type: ClusterIP
```

## nginx Security Configuration

### Production nginx Configuration

**File**: `nginx/nginx.prod.conf`

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

    # Content Security Policy
    add_header Content-Security-Policy "
        default-src 'self';
        script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdnjs.cloudflare.com https://unpkg.com;
        style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdnjs.cloudflare.com;
        font-src 'self' https://fonts.gstatic.com https://cdnjs.cloudflare.com;
        img-src 'self' data: https: blob:;
        connect-src 'self' wss: https://api.dealsphere.com;
        media-src 'self';
        object-src 'none';
        child-src 'self';
        frame-ancestors 'none';
        base-uri 'self';
        form-action 'self';
        upgrade-insecure-requests;
    " always;

    # Rate limiting zones
    limit_req_zone $binary_remote_addr zone=auth:10m rate=5r/m;
    limit_req_zone $binary_remote_addr zone=api:10m rate=30r/m;
    limit_req_zone $binary_remote_addr zone=reset:10m rate=1r/m;
    limit_req_zone $binary_remote_addr zone=static:10m rate=50r/m;

    # Connection limiting
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

    # CORS origin mapping
    map $http_origin $allowed_origin {
        default "";
        "https://app.dealsphere.com" $http_origin;
        "https://admin.dealsphere.com" $http_origin;
    }

    # Upstream backend
    upstream backend {
        server dealsphere-backend:8080 max_fails=3 fail_timeout=30s;
        keepalive 32;
    }

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Main server block
    server {
        listen 443 ssl http2;
        server_name api.dealsphere.com;

        ssl_certificate /etc/nginx/ssl/api.dealsphere.com.crt;
        ssl_certificate_key /etc/nginx/ssl/api.dealsphere.com.key;

        # Security
        limit_conn conn_limit_per_ip 20;

        # Authentication endpoints - strict rate limiting
        location /api/auth/login {
            limit_req zone=auth burst=10 nodelay;
            limit_req_status 429;

            # CORS
            add_header 'Access-Control-Allow-Origin' '$allowed_origin' always;
            add_header 'Access-Control-Allow-Methods' 'POST, OPTIONS' always;
            add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Authorization' always;
            add_header 'Access-Control-Allow-Credentials' 'true' always;
            add_header 'Access-Control-Max-Age' '86400' always;

            if ($request_method = OPTIONS) {
                return 204;
            }

            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_connect_timeout 30s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
        }

        # Password reset - very strict rate limiting
        location /api/auth/request-password-reset {
            limit_req zone=reset burst=2 nodelay;
            limit_req_status 429;

            # CORS
            add_header 'Access-Control-Allow-Origin' '$allowed_origin' always;
            add_header 'Access-Control-Allow-Methods' 'POST, OPTIONS' always;
            add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept' always;
            add_header 'Access-Control-Allow-Credentials' 'true' always;

            if ($request_method = OPTIONS) {
                return 204;
            }

            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # General API endpoints
        location /api/ {
            limit_req zone=api burst=50 nodelay;
            limit_req_status 429;

            # CORS
            add_header 'Access-Control-Allow-Origin' '$allowed_origin' always;
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
            add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Authorization' always;
            add_header 'Access-Control-Allow-Credentials' 'true' always;
            add_header 'Access-Control-Max-Age' '86400' always;

            if ($request_method = OPTIONS) {
                return 204;
            }

            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_connect_timeout 30s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 4k;
        }

        # Health checks
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }

    # Frontend server
    server {
        listen 443 ssl http2;
        server_name app.dealsphere.com;

        ssl_certificate /etc/nginx/ssl/app.dealsphere.com.crt;
        ssl_certificate_key /etc/nginx/ssl/app.dealsphere.com.key;

        root /var/www/html;
        index index.html;

        # Static files with caching
        location /static/ {
            limit_req zone=static burst=100 nodelay;
            expires 1y;
            add_header Cache-Control "public, immutable";
            add_header X-Content-Type-Options "nosniff";
        }

        # SPA routing
        location / {
            try_files $uri $uri/ /index.html;
            expires 1h;
            add_header Cache-Control "public, no-cache, must-revalidate";
        }
    }

    # HTTP to HTTPS redirect
    server {
        listen 80;
        server_name api.dealsphere.com app.dealsphere.com;
        return 301 https://$server_name$request_uri;
    }
}
```

## Database Setup

### PostgreSQL Production Configuration

**File**: `database/postgresql.prod.conf`

```sql
# PostgreSQL production configuration for authentication system

# Connection settings
listen_addresses = '*'
port = 5432
max_connections = 100
shared_buffers = 256MB
effective_cache_size = 1GB
work_mem = 4MB
maintenance_work_mem = 64MB

# WAL settings
wal_level = replica
max_wal_senders = 3
checkpoint_completion_target = 0.9
wal_buffers = 16MB
checkpoint_segments = 32

# Logging
log_destination = 'stderr'
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_statement = 'ddl'
log_min_duration_statement = 1000
log_connections = on
log_disconnections = on

# Security
ssl = on
ssl_cert_file = '/etc/ssl/certs/server.crt'
ssl_key_file = '/etc/ssl/private/server.key'
password_encryption = scram-sha-256

# Performance
random_page_cost = 1.1
seq_page_cost = 1.0
default_statistics_target = 100
```

### Database Migration Script

**File**: `database/migrate.sh`

```bash
#!/bin/bash
set -e

# Database migration script for production deployment

# Configuration
DB_HOST=${DB_HOST:-localhost}
DB_PORT=${DB_PORT:-5432}
DB_NAME=${DB_NAME:-dealsphere_prod}
DB_USER=${DB_USER:-dealsphere}
DB_PASSWORD=${DB_PASSWORD}

# Check if database exists
echo "Checking database connection..."
PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d postgres -c "SELECT 1" > /dev/null

# Create database if it doesn't exist
PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d postgres -tc "SELECT 1 FROM pg_database WHERE datname = '$DB_NAME'" | grep -q 1 || {
    echo "Creating database $DB_NAME..."
    PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d postgres -c "CREATE DATABASE $DB_NAME"
}

# Run migrations
echo "Running Flyway migrations..."
flyway -url=jdbc:postgresql://$DB_HOST:$DB_PORT/$DB_NAME \
       -user=$DB_USER \
       -password=$DB_PASSWORD \
       -locations=filesystem:./migrations \
       migrate

# Verify migration
echo "Verifying database structure..."
PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME -c "
SELECT
    schemaname,
    tablename,
    tableowner
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY tablename;
"

echo "Database migration completed successfully!"
```

## Security Configuration

### SSL Certificate Setup

```bash
#!/bin/bash
# SSL certificate setup script

# Create SSL directory
mkdir -p /etc/nginx/ssl

# Generate certificates using certbot (Let's Encrypt)
certbot certonly \
  --nginx \
  --email admin@dealsphere.com \
  --agree-tos \
  --no-eff-email \
  -d api.dealsphere.com \
  -d app.dealsphere.com \
  -d admin.dealsphere.com

# Set proper permissions
chown -R nginx:nginx /etc/nginx/ssl
chmod 600 /etc/nginx/ssl/*.key
chmod 644 /etc/nginx/ssl/*.crt

# Setup auto-renewal
echo "0 2 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx" | crontab -
```

### JWT Security Configuration

**File**: `security/jwt-setup.sh`

```bash
#!/bin/bash
# JWT security setup

# Generate secure JWT secret (256-bit)
JWT_SECRET=$(openssl rand -base64 64 | tr -d '\n')
echo "Generated JWT secret: $JWT_SECRET"

# Store in Docker secrets
echo $JWT_SECRET | docker secret create jwt_secret -

# Store in Kubernetes secret
kubectl create secret generic jwt-secret \
  --from-literal=secret=$JWT_SECRET \
  --namespace=dealsphere

# Verify secret creation
echo "JWT secret created successfully!"
```

### Database Security Setup

```bash
#!/bin/bash
# Database security setup

# Create database user with limited privileges
PGPASSWORD=$POSTGRES_PASSWORD psql -U postgres -c "
CREATE USER dealsphere WITH PASSWORD '$DB_PASSWORD';
CREATE DATABASE dealsphere_prod OWNER dealsphere;
GRANT CONNECT ON DATABASE dealsphere_prod TO dealsphere;
GRANT USAGE ON SCHEMA public TO dealsphere;
GRANT CREATE ON SCHEMA public TO dealsphere;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO dealsphere;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO dealsphere;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO dealsphere;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO dealsphere;
"

# Enable audit logging
echo "shared_preload_libraries = 'pgaudit'" >> /var/lib/postgresql/data/postgresql.conf
echo "pgaudit.log = 'all'" >> /var/lib/postgresql/data/postgresql.conf
echo "pgaudit.log_catalog = off" >> /var/lib/postgresql/data/postgresql.conf

echo "Database security configuration completed!"
```

## Deployment Process

### Automated Deployment Script

**File**: `deploy/deploy.sh`

```bash
#!/bin/bash
set -e

# Deployment script for DealSphere authentication system

# Configuration
ENVIRONMENT=${1:-production}
VERSION=${2:-latest}
HEALTH_CHECK_RETRIES=30
HEALTH_CHECK_INTERVAL=10

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

warn() {
    echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] WARNING: $1${NC}"
}

error() {
    echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1${NC}"
    exit 1
}

# Pre-deployment checks
log "Starting pre-deployment checks..."

# Check Docker is running
docker info >/dev/null 2>&1 || error "Docker is not running"

# Check required environment variables
required_vars=("DATABASE_URL" "JWT_SECRET" "SMTP_PASSWORD")
for var in "${required_vars[@]}"; do
    if [[ -z "${!var}" ]]; then
        error "Required environment variable $var is not set"
    fi
done

# Check SSL certificates
if [[ ! -f "/etc/nginx/ssl/api.dealsphere.com.crt" ]]; then
    warn "SSL certificate not found, generating self-signed certificate"
    # Generate self-signed certificate for testing
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout /etc/nginx/ssl/api.dealsphere.com.key \
        -out /etc/nginx/ssl/api.dealsphere.com.crt \
        -subj "/C=US/ST=State/L=City/O=DealSphere/CN=api.dealsphere.com"
fi

log "Pre-deployment checks completed successfully"

# Database migration
log "Running database migrations..."
./database/migrate.sh || error "Database migration failed"

# Pull latest images
log "Pulling latest Docker images..."
docker-compose -f docker-compose.${ENVIRONMENT}.yml pull

# Deploy with rolling update
log "Starting deployment..."

# Backend deployment
log "Deploying backend service..."
docker-compose -f docker-compose.${ENVIRONMENT}.yml up -d --no-deps backend

# Wait for backend health check
log "Waiting for backend to be healthy..."
for i in $(seq 1 $HEALTH_CHECK_RETRIES); do
    if curl -f http://localhost:8080/actuator/health >/dev/null 2>&1; then
        log "Backend is healthy"
        break
    fi
    if [[ $i -eq $HEALTH_CHECK_RETRIES ]]; then
        error "Backend health check failed after $HEALTH_CHECK_RETRIES attempts"
    fi
    warn "Backend not ready yet, attempt $i/$HEALTH_CHECK_RETRIES"
    sleep $HEALTH_CHECK_INTERVAL
done

# Frontend deployment
log "Deploying frontend service..."
docker-compose -f docker-compose.${ENVIRONMENT}.yml up -d --no-deps frontend

# Nginx deployment
log "Deploying nginx service..."
docker-compose -f docker-compose.${ENVIRONMENT}.yml up -d --no-deps nginx

# Final health checks
log "Running post-deployment health checks..."

# Test authentication endpoint
test_auth_endpoint() {
    local response=$(curl -s -o /dev/null -w "%{http_code}" \
        -X POST \
        -H "Content-Type: application/json" \
        -d '{"query":"query { __schema { types { name } } }"}' \
        https://api.dealsphere.com/graphql)

    if [[ "$response" == "200" ]]; then
        log "GraphQL endpoint is responding correctly"
        return 0
    else
        error "GraphQL endpoint health check failed (HTTP $response)"
    fi
}

test_auth_endpoint

# Test frontend
test_frontend() {
    local response=$(curl -s -o /dev/null -w "%{http_code}" https://app.dealsphere.com)
    if [[ "$response" == "200" ]]; then
        log "Frontend is responding correctly"
        return 0
    else
        error "Frontend health check failed (HTTP $response)"
    fi
}

test_frontend

# Cleanup old containers
log "Cleaning up old containers..."
docker system prune -f

log "Deployment completed successfully!"
log "Services are available at:"
log "  - Frontend: https://app.dealsphere.com"
log "  - API: https://api.dealsphere.com"
log "  - GraphQL: https://api.dealsphere.com/graphql"

# Store deployment metadata
cat > deployment-info.json << EOF
{
  "deployment_time": "$(date -Iseconds)",
  "environment": "$ENVIRONMENT",
  "version": "$VERSION",
  "services": {
    "backend": "healthy",
    "frontend": "healthy",
    "nginx": "healthy",
    "database": "healthy"
  }
}
EOF

log "Deployment metadata saved to deployment-info.json"
```

### Rollback Script

**File**: `deploy/rollback.sh`

```bash
#!/bin/bash
set -e

# Rollback script for DealSphere authentication system

PREVIOUS_VERSION=${1}
if [[ -z "$PREVIOUS_VERSION" ]]; then
    echo "Usage: $0 <previous_version>"
    echo "Example: $0 v1.2.3"
    exit 1
fi

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1"
    exit 1
}

log "Starting rollback to version $PREVIOUS_VERSION"

# Stop current services
log "Stopping current services..."
docker-compose -f docker-compose.production.yml down

# Deploy previous version
log "Deploying previous version..."
VERSION=$PREVIOUS_VERSION docker-compose -f docker-compose.production.yml up -d

# Wait for services to be healthy
log "Waiting for services to be healthy..."
sleep 30

# Health check
curl -f http://localhost:8080/actuator/health || error "Rollback health check failed"

log "Rollback to version $PREVIOUS_VERSION completed successfully"
```

## Monitoring & Observability

### Application Health Checks

**File**: `monitoring/health-checks.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: health-checks
data:
  health-check.sh: |
    #!/bin/bash

    # Authentication service health checks

    # Check backend health
    if ! curl -f http://backend:8080/actuator/health >/dev/null 2>&1; then
        echo "CRITICAL: Backend service is down"
        exit 2
    fi

    # Check database connectivity
    if ! PGPASSWORD=$DB_PASSWORD psql -h database -U dealsphere -d dealsphere_prod -c "SELECT 1" >/dev/null 2>&1; then
        echo "CRITICAL: Database is unreachable"
        exit 2
    fi

    # Check authentication functionality
    auth_response=$(curl -s -X POST \
        -H "Content-Type: application/json" \
        -d '{"query":"query { __schema { types { name } } }"}' \
        http://backend:8080/graphql)

    if [[ $? -ne 0 ]]; then
        echo "WARNING: Authentication endpoint not responding"
        exit 1
    fi

    # Check email service connectivity
    if ! nc -z $SMTP_HOST $SMTP_PORT; then
        echo "WARNING: SMTP service unreachable"
        exit 1
    fi

    echo "OK: All authentication services are healthy"
    exit 0
```

### Security Monitoring

**File**: `monitoring/security-alerts.yaml`

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: dealsphere-security-alerts
spec:
  groups:
  - name: dealsphere.security
    rules:
    - alert: HighFailedLoginRate
      expr: rate(security_events_total{event_type="LOGIN_FAILURE"}[5m]) > 10
      for: 5m
      labels:
        severity: warning
        service: authentication
      annotations:
        summary: "High failed login rate detected"
        description: "{{ $value }} failed login attempts per second in the last 5 minutes"

    - alert: SQLInjectionAttempt
      expr: increase(security_events_total{event_type="SQL_INJECTION_ATTEMPT"}[1m]) > 0
      for: 0s
      labels:
        severity: critical
        service: authentication
      annotations:
        summary: "SQL injection attempt detected"
        description: "SQL injection attempt from {{ $labels.ip_address }}"

    - alert: BruteForceAttack
      expr: rate(security_events_total{event_type="BRUTE_FORCE_ATTEMPT"}[1m]) > 1
      for: 2m
      labels:
        severity: critical
        service: authentication
      annotations:
        summary: "Brute force attack detected"
        description: "Brute force attack from {{ $labels.ip_address }}"

    - alert: UnauthorizedAccess
      expr: rate(security_events_total{event_type="ACCESS_DENIED"}[5m]) > 5
      for: 5m
      labels:
        severity: warning
        service: authentication
      annotations:
        summary: "High unauthorized access attempts"
        description: "{{ $value }} unauthorized access attempts per second"
```

## Backup & Recovery

### Database Backup Script

**File**: `backup/db-backup.sh`

```bash
#!/bin/bash
set -e

# Database backup script for production

BACKUP_DIR="/backups/database"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="dealsphere_prod_$DATE.sql"
RETENTION_DAYS=30

# Create backup directory
mkdir -p $BACKUP_DIR

# Create full database backup
log "Creating database backup..."
PGPASSWORD=$DB_PASSWORD pg_dump \
    -h $DB_HOST \
    -p $DB_PORT \
    -U $DB_USER \
    -d $DB_NAME \
    --no-password \
    --verbose \
    --format=custom \
    --compress=9 \
    > $BACKUP_DIR/$BACKUP_FILE

# Verify backup
if [[ -f "$BACKUP_DIR/$BACKUP_FILE" && -s "$BACKUP_DIR/$BACKUP_FILE" ]]; then
    log "Database backup created successfully: $BACKUP_FILE"
else
    error "Database backup failed"
fi

# Upload to cloud storage (AWS S3 example)
if [[ -n "$AWS_S3_BUCKET" ]]; then
    aws s3 cp $BACKUP_DIR/$BACKUP_FILE s3://$AWS_S3_BUCKET/database-backups/
    log "Backup uploaded to S3"
fi

# Cleanup old backups
find $BACKUP_DIR -name "dealsphere_prod_*.sql" -mtime +$RETENTION_DAYS -delete
log "Old backups cleaned up (older than $RETENTION_DAYS days)"
```

### Disaster Recovery Plan

**File**: `disaster-recovery/recovery-plan.md`

```markdown
# Authentication System Disaster Recovery Plan

## Recovery Time Objectives (RTO)
- **Critical**: 15 minutes (authentication down)
- **High**: 1 hour (degraded performance)
- **Medium**: 4 hours (non-critical features)

## Recovery Point Objectives (RPO)
- **Database**: 15 minutes (continuous backup)
- **Application**: 1 hour (latest deployment)
- **Configuration**: 5 minutes (version controlled)

## Recovery Procedures

### Complete System Failure
1. **Assess Impact**: Determine scope of failure
2. **Activate DR Site**: Switch to backup infrastructure
3. **Restore Database**: From latest backup
4. **Deploy Services**: Using stored configuration
5. **Verify Functionality**: Run health checks
6. **Update DNS**: Point to DR infrastructure

### Database Failure
1. **Stop Application**: Prevent data corruption
2. **Restore from Backup**: Latest full backup
3. **Apply Transaction Logs**: If available
4. **Restart Services**: In correct order
5. **Verify Data Integrity**: Run consistency checks

### Security Incident
1. **Isolate Systems**: Prevent further damage
2. **Assess Damage**: Determine what was compromised
3. **Reset Credentials**: All JWT secrets, passwords
4. **Audit Logs**: Review security events
5. **Restore Clean State**: From known good backup
6. **Implement Fixes**: Address vulnerabilities
```

## Operation Procedures

### Daily Operations Checklist

```bash
#!/bin/bash
# Daily operations checklist for authentication system

echo "DealSphere Authentication System - Daily Operations Checklist"
echo "Date: $(date)"
echo

# Check service health
echo "1. Service Health Check"
curl -f https://api.dealsphere.com/health && echo "✓ API Health OK" || echo "✗ API Health FAILED"
curl -f https://app.dealsphere.com && echo "✓ Frontend OK" || echo "✗ Frontend FAILED"

# Check database
echo "2. Database Health"
PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -U $DB_USER -d $DB_NAME -c "SELECT COUNT(*) FROM user_profiles;" && echo "✓ Database OK" || echo "✗ Database FAILED"

# Check disk space
echo "3. Disk Space Check"
df -h | grep -E "(8[0-9]|9[0-9])%" && echo "⚠ High disk usage detected" || echo "✓ Disk space OK"

# Check logs for errors
echo "4. Error Log Check"
error_count=$(docker logs dealsphere-backend 2>&1 | grep -c ERROR || true)
if [[ $error_count -gt 10 ]]; then
    echo "⚠ High error count: $error_count errors in backend logs"
else
    echo "✓ Error logs within normal range ($error_count errors)"
fi

# Check SSL certificates
echo "5. SSL Certificate Check"
expiry=$(openssl x509 -in /etc/nginx/ssl/api.dealsphere.com.crt -noout -enddate | cut -d= -f2)
expiry_epoch=$(date -d "$expiry" +%s)
current_epoch=$(date +%s)
days_until_expiry=$(( ($expiry_epoch - $current_epoch) / 86400 ))

if [[ $days_until_expiry -lt 30 ]]; then
    echo "⚠ SSL certificate expires in $days_until_expiry days"
else
    echo "✓ SSL certificate valid for $days_until_expiry days"
fi

# Check backup status
echo "6. Backup Status Check"
latest_backup=$(ls -t /backups/database/dealsphere_prod_*.sql | head -1)
if [[ -f "$latest_backup" ]]; then
    backup_age=$(( ($(date +%s) - $(stat -c %Y "$latest_backup")) / 3600 ))
    if [[ $backup_age -lt 24 ]]; then
        echo "✓ Latest backup is $backup_age hours old"
    else
        echo "⚠ Latest backup is $backup_age hours old (>24h)"
    fi
else
    echo "✗ No backup found"
fi

echo
echo "Daily operations checklist completed"
```

This comprehensive deployment guide ensures secure, reliable, and scalable deployment of the DealSphere authentication system in production environments.