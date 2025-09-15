# Production Deployment Guide

This guide provides step-by-step instructions for deploying the DealSphere platform to production environments.

## Overview

The DealSphere production deployment consists of:

- **Backend**: Spring Boot application with PostgreSQL database
- **Frontend**: React application served via nginx
- **Observability**: Prometheus, Grafana, Loki, Jaeger stack
- **Infrastructure**: Containerized services with Docker/Kubernetes
- **Security**: SSL/TLS, secrets management, network security

## Prerequisites

### Infrastructure Requirements

**Minimum Production Specifications:**
```
Backend Service:
- CPU: 2 vCPUs
- RAM: 4GB
- Storage: 50GB SSD

Database (PostgreSQL):
- CPU: 2 vCPUs
- RAM: 4GB
- Storage: 100GB SSD (with backup)

Frontend/Nginx:
- CPU: 1 vCPU
- RAM: 1GB
- Storage: 20GB SSD

Monitoring Stack:
- CPU: 2 vCPUs
- RAM: 4GB
- Storage: 100GB SSD
```

**Recommended Production Specifications:**
```
Backend Service:
- CPU: 4 vCPUs
- RAM: 8GB
- Storage: 100GB SSD

Database (PostgreSQL):
- CPU: 4 vCPUs
- RAM: 8GB
- Storage: 500GB SSD (with automated backup)

Load Balancer/Frontend:
- CPU: 2 vCPUs
- RAM: 2GB
- Storage: 50GB SSD

Monitoring Stack:
- CPU: 4 vCPUs
- RAM: 8GB
- Storage: 200GB SSD
```

### Software Requirements

- **Docker**: 24.0+
- **Docker Compose**: 2.20+
- **Domain**: Registered domain with DNS control
- **SSL Certificate**: Valid SSL certificate (Let's Encrypt recommended)

## Deployment Methods

### Method 1: Docker Compose (Recommended for Small Teams)

#### 1. Server Preparation

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Reboot to apply group changes
sudo reboot
```

#### 2. Application Deployment

```bash
# Clone the repository
git clone https://github.com/your-org/dealsphere-platform.git
cd dealsphere-platform

# Create production environment file
cp .env.production .env

# Edit environment variables (see Environment Configuration section)
nano .env

# Pull latest images
docker compose pull

# Start the application stack
docker compose -f docker-compose.yml up -d

# Verify deployment
docker compose ps
docker compose logs -f backend
```

#### 3. Database Setup

```bash
# Check database initialization
docker compose exec backend ./gradlew flyway:info

# Run database migrations (if needed)
docker compose exec backend ./gradlew flyway:migrate

# Verify database connection
docker compose exec db psql -U $POSTGRES_USER -d $POSTGRES_DB -c "SELECT version();"
```

### Method 2: Kubernetes (Recommended for Scalability)

#### 1. Kubernetes Manifests

Create Kubernetes deployment files:

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dealsphere-prod
---
# k8s/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dealsphere-backend
  namespace: dealsphere-prod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dealsphere-backend
  template:
    metadata:
      labels:
        app: dealsphere-backend
    spec:
      containers:
      - name: backend
        image: your-registry/dealsphere-backend:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
        - name: POSTGRES_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
---
apiVersion: v1
kind: Service
metadata:
  name: dealsphere-backend-service
  namespace: dealsphere-prod
spec:
  selector:
    app: dealsphere-backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

#### 2. Deploy to Kubernetes

```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/

# Check deployment status
kubectl get pods -n dealsphere-prod
kubectl get services -n dealsphere-prod

# View logs
kubectl logs -f deployment/dealsphere-backend -n dealsphere-prod
```

## Environment Configuration

### Production Environment Variables

Create `.env.production`:

```bash
# === Database Configuration ===
POSTGRES_DB=dealsphere_prod
POSTGRES_USER=dealsphere_prod
POSTGRES_PASSWORD=your-secure-db-password
POSTGRES_HOST=your-db-host.amazonaws.com
POSTGRES_PORT=5432

# === Application Configuration ===
SPRING_PROFILES_ACTIVE=production
SERVER_PORT=8080
SPRING_JPA_HIBERNATE_DDL_AUTO=validate

# === Security Configuration ===
JWT_SECRET=your-256-bit-jwt-secret-key-here
JWT_EXPIRATION_HOURS=24
SSL_ENABLED=true

# === Frontend Configuration ===
VITE_API_URL=https://api.yourdomain.com
VITE_GRAPHQL_URL=https://api.yourdomain.com/graphql
VITE_ENV=production

# === CORS Configuration ===
CORS_ALLOWED_ORIGINS=https://yourdomain.com,https://www.yourdomain.com
CORS_ALLOWED_METHODS=GET,POST,OPTIONS,PUT,DELETE
CORS_ALLOWED_HEADERS=*
CORS_ALLOW_CREDENTIALS=true

# === Logging Configuration ===
LOGGING_LEVEL_ROOT=WARN
LOGGING_LEVEL_COM_DEALSPHERE=INFO

# === Monitoring Configuration ===
MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE=health,metrics,prometheus,info
MANAGEMENT_ENDPOINT_HEALTH_SHOW_DETAILS=when-authorized

# === Email Configuration (Optional) ===
SPRING_MAIL_HOST=smtp.yourdomain.com
SPRING_MAIL_PORT=587
SPRING_MAIL_USERNAME=noreply@yourdomain.com
SPRING_MAIL_PASSWORD=your-email-password
SPRING_MAIL_PROPERTIES_MAIL_SMTP_AUTH=true
SPRING_MAIL_PROPERTIES_MAIL_SMTP_STARTTLS_ENABLE=true
```

## SSL/TLS Configuration

### Option 1: Let's Encrypt with Certbot

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Verify auto-renewal
sudo certbot renew --dry-run
```

### Option 2: Custom SSL Certificate

```bash
# Create SSL directory
sudo mkdir -p /etc/ssl/dealsphere

# Copy your certificate files
sudo cp your-domain.crt /etc/ssl/dealsphere/
sudo cp your-domain.key /etc/ssl/dealsphere/
sudo cp ca-bundle.crt /etc/ssl/dealsphere/

# Set proper permissions
sudo chmod 600 /etc/ssl/dealsphere/*.key
sudo chmod 644 /etc/ssl/dealsphere/*.crt
```

### Nginx SSL Configuration

```nginx
# /etc/nginx/sites-available/dealsphere
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/ssl/dealsphere/your-domain.crt;
    ssl_certificate_key /etc/ssl/dealsphere/your-domain.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        proxy_pass http://frontend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Database Management

### Production Database Setup

#### PostgreSQL Production Configuration

```sql
-- Create production database and user
CREATE DATABASE dealsphere_prod;
CREATE USER dealsphere_prod WITH ENCRYPTED PASSWORD 'your-secure-password';
GRANT ALL PRIVILEGES ON DATABASE dealsphere_prod TO dealsphere_prod;
GRANT ALL ON SCHEMA public TO dealsphere_prod;
```

#### Database Migration Strategy

```bash
# 1. Backup current database (if updating)
pg_dump -h your-db-host -U dealsphere_prod dealsphere_prod > backup_$(date +%Y%m%d_%H%M%S).sql

# 2. Test migrations on staging first
docker compose -f docker-compose.staging.yml exec backend ./gradlew flyway:info
docker compose -f docker-compose.staging.yml exec backend ./gradlew flyway:migrate

# 3. Apply to production (during maintenance window)
docker compose exec backend ./gradlew flyway:migrate

# 4. Verify migration success
docker compose exec backend ./gradlew flyway:info
```

#### Database Backup Strategy

```bash
# Create backup script
cat > /scripts/backup-dealsphere.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backups/dealsphere"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="dealsphere_prod_backup_${TIMESTAMP}.sql"

mkdir -p $BACKUP_DIR

# Create backup
pg_dump -h your-db-host -U dealsphere_prod dealsphere_prod > $BACKUP_DIR/$BACKUP_FILE

# Compress backup
gzip $BACKUP_DIR/$BACKUP_FILE

# Keep only last 30 days of backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_FILE.gz"
EOF

chmod +x /scripts/backup-dealsphere.sh

# Schedule daily backups
echo "0 2 * * * /scripts/backup-dealsphere.sh" | sudo crontab -
```

## Health Checks and Monitoring

### Application Health Endpoints

```bash
# Check application health
curl -f https://yourdomain.com/api/actuator/health

# Check detailed health information
curl -f https://yourdomain.com/api/actuator/health/db
curl -f https://yourdomain.com/api/actuator/health/diskSpace

# Check metrics
curl -f https://yourdomain.com/api/actuator/metrics
curl -f https://yourdomain.com/api/actuator/prometheus
```

### Production Monitoring Setup

```bash
# Start monitoring stack
cd monitoring
docker compose -f docker-compose.monitoring.yml up -d

# Configure alerts for production
# See: observability-monitoring.md for detailed setup
```

### Load Balancer Health Checks

Configure your load balancer health checks:
- **Health Check URL**: `https://yourdomain.com/api/actuator/health`
- **Interval**: 30 seconds
- **Timeout**: 10 seconds
- **Healthy Threshold**: 2 consecutive successes
- **Unhealthy Threshold**: 3 consecutive failures

## Scaling and Performance

### Horizontal Scaling

#### Docker Compose Scaling

```bash
# Scale backend service
docker compose up -d --scale backend=3

# Use nginx load balancing
# Update nginx.conf:
upstream backend {
    server backend_1:8080;
    server backend_2:8080;
    server backend_3:8080;
}
```

#### Kubernetes Scaling

```bash
# Scale deployment
kubectl scale deployment dealsphere-backend --replicas=5 -n dealsphere-prod

# Auto-scaling configuration
kubectl apply -f - <<EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: dealsphere-backend-hpa
  namespace: dealsphere-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: dealsphere-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
EOF
```

### Performance Optimization

#### JVM Tuning

```bash
# Production JVM settings
JAVA_OPTS="-Xms2g -Xmx4g \
-XX:+UseG1GC \
-XX:MaxGCPauseMillis=200 \
-XX:+HeapDumpOnOutOfMemoryError \
-XX:HeapDumpPath=/tmp/heapdump.hprof \
-Djava.security.egd=file:/dev/./urandom"
```

#### Database Performance

```sql
-- PostgreSQL performance tuning
ALTER SYSTEM SET shared_buffers = '256MB';
ALTER SYSTEM SET effective_cache_size = '1GB';
ALTER SYSTEM SET maintenance_work_mem = '64MB';
ALTER SYSTEM SET checkpoint_completion_target = 0.7;
ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM SET default_statistics_target = 100;
ALTER SYSTEM SET random_page_cost = 1.1;
ALTER SYSTEM SET effective_io_concurrency = 200;

-- Reload configuration
SELECT pg_reload_conf();
```

## Disaster Recovery

### Backup Strategy

**Automated Backups:**
- **Database**: Daily full backup + continuous WAL archiving
- **Application Data**: Weekly backup of uploaded files
- **Configuration**: Version-controlled infrastructure as code

**Backup Locations:**
- **Primary**: Local server storage
- **Secondary**: Cloud storage (S3, GCS, Azure Blob)
- **Tertiary**: Geographic backup location

### Recovery Procedures

#### Database Recovery

```bash
# Stop application
docker compose stop backend

# Restore database from backup
gunzip -c /backups/dealsphere/dealsphere_prod_backup_20231215_020000.sql.gz | \
  psql -h your-db-host -U dealsphere_prod dealsphere_prod

# Restart application
docker compose start backend

# Verify recovery
curl -f https://yourdomain.com/api/actuator/health
```

#### Full System Recovery

```bash
# 1. Provision new infrastructure
# 2. Restore application code
git clone https://github.com/your-org/dealsphere-platform.git

# 3. Restore configuration
cp /backups/config/.env.production .env

# 4. Restore database
# (See database recovery above)

# 5. Deploy application
docker compose -f docker-compose.yml up -d

# 6. Verify all services
./scripts/health-check.sh
```

## Security Hardening

### Server Security

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Configure firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable

# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl reload sshd

# Install fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Application Security

- **Secrets Management**: Use environment variables, never commit secrets
- **Input Validation**: All user inputs validated and sanitized
- **Authentication**: JWT tokens with proper expiration
- **Authorization**: Role-based access control (RBAC)
- **HTTPS**: All communication encrypted with TLS 1.2+
- **Security Headers**: HSTS, X-Frame-Options, CSP implemented

### Network Security

```bash
# Docker network isolation
docker network create --driver bridge dealsphere-network

# Configure iptables rules (example)
sudo iptables -A INPUT -p tcp --dport 8080 -s 10.0.0.0/8 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP
```

## Troubleshooting

### Common Production Issues

#### Application Won't Start

```bash
# Check container logs
docker compose logs backend
docker compose logs frontend
docker compose logs db

# Check resource usage
docker stats

# Check disk space
df -h

# Check memory usage
free -h

# Check application health
curl -f https://yourdomain.com/api/actuator/health
```

#### Database Connection Issues

```bash
# Test database connectivity
docker compose exec backend ping db
docker compose exec backend nc -zv db 5432

# Check database logs
docker compose logs db

# Verify database credentials
docker compose exec backend env | grep POSTGRES
```

#### High Memory Usage

```bash
# Monitor JVM memory
curl -s https://yourdomain.com/api/actuator/metrics/jvm.memory.used | jq .

# Generate heap dump
docker compose exec backend kill -3 1

# Analyze garbage collection
curl -s https://yourdomain.com/api/actuator/metrics/jvm.gc.pause | jq .
```

#### SSL Certificate Issues

```bash
# Check certificate expiration
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com 2>/dev/null | \
  openssl x509 -noout -dates

# Renew Let's Encrypt certificate
sudo certbot renew --force-renewal

# Test SSL configuration
curl -I https://yourdomain.com
```

### Performance Issues

#### High CPU Usage

```bash
# Monitor CPU usage
docker stats

# Check for long-running queries
docker compose exec db psql -U $POSTGRES_USER -d $POSTGRES_DB -c "
  SELECT pid, now() - pg_stat_activity.query_start AS duration, query
  FROM pg_stat_activity
  WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';
"

# Profile application
curl -s https://yourdomain.com/api/actuator/metrics/process.cpu.usage | jq .
```

#### High Response Times

```bash
# Check response times
curl -w "@curl-format.txt" -o /dev/null -s https://yourdomain.com/api/health

# Monitor database performance
docker compose exec db psql -U $POSTGRES_USER -d $POSTGRES_DB -c "
  SELECT query, mean_time, calls
  FROM pg_stat_statements
  ORDER BY mean_time DESC
  LIMIT 10;
"

# Check for thread pool exhaustion
curl -s https://yourdomain.com/api/actuator/metrics/http.server.requests | jq .
```

## Maintenance

### Regular Maintenance Tasks

**Daily:**
- Monitor application logs
- Check system resource usage
- Verify backup completion
- Review security alerts

**Weekly:**
- Update system packages
- Review application metrics
- Test disaster recovery procedures
- Clean up old log files

**Monthly:**
- Update application dependencies
- Review and rotate secrets
- Performance optimization analysis
- Security vulnerability scanning

### Maintenance Windows

```bash
# Create maintenance script
cat > /scripts/maintenance.sh << 'EOF'
#!/bin/bash
echo "Starting maintenance window..."

# 1. Enable maintenance mode
docker compose exec nginx cp /etc/nginx/maintenance.html /usr/share/nginx/html/index.html

# 2. Graceful shutdown
docker compose stop backend
sleep 30

# 3. Backup database
/scripts/backup-dealsphere.sh

# 4. Update application
git pull origin main
docker compose pull

# 5. Database migrations
docker compose up -d db
sleep 30
docker compose run --rm backend ./gradlew flyway:migrate

# 6. Start application
docker compose up -d

# 7. Health checks
sleep 60
curl -f https://yourdomain.com/api/actuator/health || exit 1

# 8. Disable maintenance mode
docker compose exec nginx cp /etc/nginx/index.html /usr/share/nginx/html/index.html

echo "Maintenance window completed successfully"
EOF

chmod +x /scripts/maintenance.sh
```

## Related Documentation

- [Cloud Infrastructure Setup](cloud-infrastructure.md)
- [Environment Management](environment-management.md)
- [Observability and Monitoring](observability-monitoring.md)
- [Security and Secrets Management](security-secrets-management.md)
- [Local CI/CD Setup](../development/local-cicd-setup.md)
- [GitHub Actions](../development/github-actions.md)