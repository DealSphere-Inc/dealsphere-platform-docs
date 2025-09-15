# Cloud Infrastructure Setup Guide

This guide provides comprehensive instructions for setting up cloud infrastructure for the DealSphere platform across major cloud providers.

## Overview

The DealSphere platform can be deployed on multiple cloud providers with Infrastructure as Code (IaC) for consistent, reproducible deployments.

**Supported Cloud Providers:**
- Amazon Web Services (AWS)
- Google Cloud Platform (GCP)
- Microsoft Azure
- Digital Ocean (simplified setup)

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │────│  Frontend (CDN) │────│   Static Assets │
│   (ALB/CLB)     │    │    (Nginx)      │    │      (S3)       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │
         │              ┌─────────────────┐
         │              │   Backend API   │
         │              │ (ECS/GKE/AKS)   │
         │              └─────────────────┘
         │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Monitoring    │    │    Database     │    │ Secrets Manager │
│ (Prometheus/    │    │ (RDS/CloudSQL/  │    │  (Vault/KMS/    │
│  Grafana)       │    │  Azure DB)      │    │   Key Vault)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Amazon Web Services (AWS)

### Prerequisites

**Required AWS CLI and Tools:**
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Install Terraform
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Configure AWS credentials
aws configure
```

### Terraform Infrastructure

#### Main Infrastructure (main.tf)

```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "dealsphere-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-west-2"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = "DealSphere"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# Variables
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-west-2"
}

variable "environment" {
  description = "Environment (staging/production)"
  type        = string
  default     = "production"
}

variable "domain_name" {
  description = "Domain name for the application"
  type        = string
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

# VPC and Networking
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "dealsphere-${var.environment}-vpc"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "dealsphere-${var.environment}-igw"
  }
}

resource "aws_subnet" "public" {
  count = 2

  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "dealsphere-${var.environment}-public-${count.index + 1}"
    Type = "Public"
  }
}

resource "aws_subnet" "private" {
  count = 2

  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "dealsphere-${var.environment}-private-${count.index + 1}"
    Type = "Private"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "dealsphere-${var.environment}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# NAT Gateway for private subnets
resource "aws_eip" "nat" {
  domain = "vpc"

  tags = {
    Name = "dealsphere-${var.environment}-nat-eip"
  }
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id

  tags = {
    Name = "dealsphere-${var.environment}-nat-gw"
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "dealsphere-${var.environment}-private-rt"
  }
}

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)

  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```

#### Database (rds.tf)

```hcl
# RDS Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "dealsphere-${var.environment}-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id

  tags = {
    Name = "dealsphere-${var.environment}-db-subnet-group"
  }
}

# Database Security Group
resource "aws_security_group" "database" {
  name_prefix = "dealsphere-${var.environment}-db-"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "PostgreSQL from application"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.backend.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "dealsphere-${var.environment}-db-sg"
  }
}

# RDS Instance
resource "aws_db_instance" "main" {
  identifier = "dealsphere-${var.environment}-db"

  engine         = "postgres"
  engine_version = "16.1"
  instance_class = var.environment == "production" ? "db.t3.medium" : "db.t3.micro"

  allocated_storage     = var.environment == "production" ? 100 : 20
  max_allocated_storage = var.environment == "production" ? 1000 : 100
  storage_type          = "gp3"
  storage_encrypted     = true

  db_name  = "dealsphere"
  username = "dealsphere"
  password = random_password.db_password.result

  vpc_security_group_ids = [aws_security_group.database.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  backup_retention_period = var.environment == "production" ? 30 : 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"

  skip_final_snapshot = var.environment != "production"
  deletion_protection = var.environment == "production"

  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn        = aws_iam_role.rds_monitoring.arn

  tags = {
    Name = "dealsphere-${var.environment}-db"
  }
}

# Random password for database
resource "random_password" "db_password" {
  length  = 32
  special = true
}

# Store database password in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "dealsphere/${var.environment}/db-password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db_password.result
}

# RDS Monitoring Role
resource "aws_iam_role" "rds_monitoring" {
  name = "dealsphere-${var.environment}-rds-monitoring-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "monitoring.rds.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "rds_monitoring" {
  role       = aws_iam_role.rds_monitoring.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonRDSEnhancedMonitoringRole"
}
```

#### ECS Cluster (ecs.tf)

```hcl
# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "dealsphere-${var.environment}"

  configuration {
    execute_command_configuration {
      logging = "OVERRIDE"
      log_configuration {
        cloud_watch_log_group_name = aws_cloudwatch_log_group.ecs.name
      }
    }
  }

  tags = {
    Name = "dealsphere-${var.environment}-cluster"
  }
}

# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "ecs" {
  name              = "/ecs/dealsphere-${var.environment}"
  retention_in_days = var.environment == "production" ? 30 : 7

  tags = {
    Name = "dealsphere-${var.environment}-logs"
  }
}

# Backend Security Group
resource "aws_security_group" "backend" {
  name_prefix = "dealsphere-${var.environment}-backend-"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "HTTP from ALB"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "dealsphere-${var.environment}-backend-sg"
  }
}

# ECS Task Definition
resource "aws_ecs_task_definition" "backend" {
  family                   = "dealsphere-${var.environment}-backend"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = var.environment == "production" ? 2048 : 1024
  memory                   = var.environment == "production" ? 4096 : 2048
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn           = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([
    {
      name  = "backend"
      image = "your-account.dkr.ecr.${var.aws_region}.amazonaws.com/dealsphere-backend:latest"

      portMappings = [
        {
          containerPort = 8080
          protocol      = "tcp"
        }
      ]

      environment = [
        {
          name  = "SPRING_PROFILES_ACTIVE"
          value = "production"
        },
        {
          name  = "POSTGRES_HOST"
          value = aws_db_instance.main.address
        },
        {
          name  = "POSTGRES_DB"
          value = aws_db_instance.main.db_name
        },
        {
          name  = "POSTGRES_USER"
          value = aws_db_instance.main.username
        }
      ]

      secrets = [
        {
          name      = "POSTGRES_PASSWORD"
          valueFrom = aws_secretsmanager_secret.db_password.arn
        }
      ]

      logConfiguration = {
        logDriver = "awslogs"
        options = {
          awslogs-group         = aws_cloudwatch_log_group.ecs.name
          awslogs-region        = var.aws_region
          awslogs-stream-prefix = "backend"
        }
      }

      healthCheck = {
        command = ["CMD-SHELL", "curl -f http://localhost:8080/actuator/health || exit 1"]
        interval = 30
        timeout = 5
        retries = 3
      }

      essential = true
    }
  ])

  tags = {
    Name = "dealsphere-${var.environment}-backend-task"
  }
}

# ECS Service
resource "aws_ecs_service" "backend" {
  name            = "dealsphere-${var.environment}-backend"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.backend.arn
  desired_count   = var.environment == "production" ? 2 : 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets         = aws_subnet.private[*].id
    security_groups = [aws_security_group.backend.id]
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.backend.arn
    container_name   = "backend"
    container_port   = 8080
  }

  depends_on = [aws_lb_listener.backend]

  tags = {
    Name = "dealsphere-${var.environment}-backend-service"
  }
}

# IAM Roles for ECS
resource "aws_iam_role" "ecs_execution" {
  name = "dealsphere-${var.environment}-ecs-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "ecs_execution" {
  role       = aws_iam_role.ecs_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

resource "aws_iam_role_policy" "ecs_secrets" {
  name = "ecs-secrets-policy"
  role = aws_iam_role.ecs_execution.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue"
        ]
        Resource = [
          aws_secretsmanager_secret.db_password.arn
        ]
      }
    ]
  })
}

resource "aws_iam_role" "ecs_task" {
  name = "dealsphere-${var.environment}-ecs-task-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}
```

#### Load Balancer (alb.tf)

```hcl
# Application Load Balancer
resource "aws_lb" "main" {
  name               = "dealsphere-${var.environment}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets           = aws_subnet.public[*].id

  enable_deletion_protection = var.environment == "production"

  tags = {
    Name = "dealsphere-${var.environment}-alb"
  }
}

# ALB Security Group
resource "aws_security_group" "alb" {
  name_prefix = "dealsphere-${var.environment}-alb-"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "dealsphere-${var.environment}-alb-sg"
  }
}

# Target Groups
resource "aws_lb_target_group" "backend" {
  name     = "dealsphere-${var.environment}-backend-tg"
  port     = 8080
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  target_type = "ip"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/actuator/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 3
  }

  tags = {
    Name = "dealsphere-${var.environment}-backend-tg"
  }
}

# SSL Certificate
data "aws_acm_certificate" "main" {
  domain   = var.domain_name
  statuses = ["ISSUED"]
}

# ALB Listeners
resource "aws_lb_listener" "redirect_http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}

resource "aws_lb_listener" "backend" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = data.aws_acm_certificate.main.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.backend.arn
  }
}
```

#### S3 and CloudFront (s3.tf)

```hcl
# S3 Bucket for Frontend
resource "aws_s3_bucket" "frontend" {
  bucket = "dealsphere-${var.environment}-frontend-${random_id.bucket_suffix.hex}"

  tags = {
    Name = "dealsphere-${var.environment}-frontend"
  }
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}

resource "aws_s3_bucket_public_access_block" "frontend" {
  bucket = aws_s3_bucket.frontend.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_versioning" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  versioning_configuration {
    status = "Enabled"
  }
}

# CloudFront Distribution
resource "aws_cloudfront_origin_access_control" "main" {
  name                              = "dealsphere-${var.environment}-oac"
  origin_access_control_origin_type = "s3"
  signing_behavior                  = "always"
  signing_protocol                  = "sigv4"
}

resource "aws_cloudfront_distribution" "main" {
  origin {
    domain_name              = aws_s3_bucket.frontend.bucket_regional_domain_name
    origin_id                = "S3-${aws_s3_bucket.frontend.bucket}"
    origin_access_control_id = aws_cloudfront_origin_access_control.main.id
  }

  origin {
    domain_name = aws_lb.main.dns_name
    origin_id   = "ALB-backend"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"
  aliases             = [var.domain_name]

  default_cache_behavior {
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.frontend.bucket}"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }

  ordered_cache_behavior {
    path_pattern     = "/api/*"
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "ALB-backend"

    forwarded_values {
      query_string = true
      headers      = ["*"]
      cookies {
        forward = "all"
      }
    }

    viewer_protocol_policy = "https-only"
    min_ttl                = 0
    default_ttl            = 0
    max_ttl                = 0
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn = data.aws_acm_certificate.main.arn
    ssl_support_method  = "sni-only"
  }

  tags = {
    Name = "dealsphere-${var.environment}-cloudfront"
  }
}

# S3 bucket policy for CloudFront
resource "aws_s3_bucket_policy" "frontend" {
  bucket = aws_s3_bucket.frontend.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Service = "cloudfront.amazonaws.com"
        }
        Action   = "s3:GetObject"
        Resource = "${aws_s3_bucket.frontend.arn}/*"
        Condition = {
          StringEquals = {
            "AWS:SourceArn" = aws_cloudfront_distribution.main.arn
          }
        }
      }
    ]
  })
}
```

### Deployment Commands

```bash
# Initialize Terraform
terraform init

# Plan deployment
terraform plan -var="domain_name=yourdomain.com"

# Apply infrastructure
terraform apply -var="domain_name=yourdomain.com"

# Get outputs
terraform output
```

## Google Cloud Platform (GCP)

### Prerequisites

```bash
# Install Google Cloud SDK
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Install Terraform
# (Same as AWS section above)

# Initialize and authenticate
gcloud init
gcloud auth application-default login
```

### Terraform Configuration

#### Main Infrastructure (main.tf)

```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }

  backend "gcs" {
    bucket = "dealsphere-terraform-state"
    prefix = "production"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
  zone    = var.zone
}

variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  description = "GCP region"
  type        = string
  default     = "us-central1"
}

variable "zone" {
  description = "GCP zone"
  type        = string
  default     = "us-central1-a"
}

variable "environment" {
  description = "Environment (staging/production)"
  type        = string
  default     = "production"
}

# VPC Network
resource "google_compute_network" "vpc" {
  name                    = "dealsphere-${var.environment}-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "public" {
  name          = "dealsphere-${var.environment}-public"
  ip_cidr_range = "10.0.1.0/24"
  region        = var.region
  network       = google_compute_network.vpc.id
}

resource "google_compute_subnetwork" "private" {
  name          = "dealsphere-${var.environment}-private"
  ip_cidr_range = "10.0.2.0/24"
  region        = var.region
  network       = google_compute_network.vpc.id

  private_ip_google_access = true
}

# Firewall Rules
resource "google_compute_firewall" "allow_http_https" {
  name    = "dealsphere-${var.environment}-allow-http-https"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["web-server"]
}

resource "google_compute_firewall" "allow_internal" {
  name    = "dealsphere-${var.environment}-allow-internal"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  source_ranges = ["10.0.0.0/8"]
}
```

#### GKE Cluster (gke.tf)

```hcl
# GKE Cluster
resource "google_container_cluster" "primary" {
  name     = "dealsphere-${var.environment}-gke"
  location = var.region

  remove_default_node_pool = true
  initial_node_count       = 1

  network    = google_compute_network.vpc.name
  subnetwork = google_compute_subnetwork.private.name

  ip_allocation_policy {
    cluster_ipv4_cidr_block  = "10.1.0.0/16"
    services_ipv4_cidr_block = "10.2.0.0/16"
  }

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  addons_config {
    http_load_balancing {
      disabled = false
    }
    horizontal_pod_autoscaling {
      disabled = false
    }
  }

  deletion_protection = var.environment == "production"
}

# Node Pool
resource "google_container_node_pool" "primary_nodes" {
  name       = "dealsphere-${var.environment}-nodes"
  location   = var.region
  cluster    = google_container_cluster.primary.name
  node_count = var.environment == "production" ? 2 : 1

  node_config {
    preemptible  = var.environment != "production"
    machine_type = var.environment == "production" ? "e2-standard-4" : "e2-standard-2"

    service_account = google_service_account.gke_nodes.email
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    labels = {
      environment = var.environment
    }

    tags = ["gke-node", "dealsphere-${var.environment}"]
  }

  autoscaling {
    min_node_count = 1
    max_node_count = var.environment == "production" ? 10 : 3
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }
}

# Service Account for GKE Nodes
resource "google_service_account" "gke_nodes" {
  account_id   = "dealsphere-${var.environment}-gke-nodes"
  display_name = "GKE Nodes Service Account"
}

resource "google_project_iam_member" "gke_nodes" {
  for_each = toset([
    "roles/logging.logWriter",
    "roles/monitoring.metricWriter",
    "roles/monitoring.viewer",
    "roles/stackdriver.resourceMetadata.writer",
    "roles/storage.objectViewer"
  ])

  project = var.project_id
  role    = each.key
  member  = "serviceAccount:${google_service_account.gke_nodes.email}"
}
```

#### Cloud SQL (cloudsql.tf)

```hcl
# Random password for database
resource "random_password" "db_password" {
  length  = 32
  special = true
}

# Cloud SQL Instance
resource "google_sql_database_instance" "main" {
  name             = "dealsphere-${var.environment}-db"
  database_version = "POSTGRES_16"
  region           = var.region

  settings {
    tier              = var.environment == "production" ? "db-standard-2" : "db-f1-micro"
    availability_type = var.environment == "production" ? "REGIONAL" : "ZONAL"
    disk_type         = "PD_SSD"
    disk_size         = var.environment == "production" ? 100 : 20
    disk_autoresize   = true

    backup_configuration {
      enabled                        = true
      start_time                     = "03:00"
      point_in_time_recovery_enabled = var.environment == "production"
      backup_retention_settings {
        retained_backups = var.environment == "production" ? 30 : 7
      }
    }

    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.vpc.id
      require_ssl     = true
    }

    maintenance_window {
      day  = 7
      hour = 3
    }

    insights_config {
      query_insights_enabled  = true
      query_string_length     = 1024
      record_application_tags = false
      record_client_address   = false
    }
  }

  deletion_protection = var.environment == "production"
}

# Database
resource "google_sql_database" "database" {
  name     = "dealsphere"
  instance = google_sql_database_instance.main.name
}

# Database User
resource "google_sql_user" "user" {
  name     = "dealsphere"
  instance = google_sql_database_instance.main.name
  password = random_password.db_password.result
}

# Private Service Access
resource "google_compute_global_address" "private_ip_address" {
  name          = "dealsphere-${var.environment}-private-ip"
  purpose       = "VPC_PEERING"
  address_type  = "INTERNAL"
  prefix_length = 16
  network       = google_compute_network.vpc.id
}

resource "google_service_networking_connection" "private_vpc_connection" {
  network                 = google_compute_network.vpc.id
  service                 = "servicenetworking.googleapis.com"
  reserved_peering_ranges = [google_compute_global_address.private_ip_address.name]
}

# Secret Manager for database credentials
resource "google_secret_manager_secret" "db_password" {
  secret_id = "dealsphere-${var.environment}-db-password"

  replication {
    automatic = true
  }
}

resource "google_secret_manager_secret_version" "db_password" {
  secret      = google_secret_manager_secret.db_password.id
  secret_data = random_password.db_password.result
}
```

### Deployment Commands

```bash
# Set project
export GOOGLE_PROJECT=your-project-id

# Initialize Terraform
terraform init

# Plan deployment
terraform plan -var="project_id=${GOOGLE_PROJECT}"

# Apply infrastructure
terraform apply -var="project_id=${GOOGLE_PROJECT}"

# Configure kubectl
gcloud container clusters get-credentials dealsphere-production-gke --region us-central1

# Deploy application
kubectl apply -f k8s/
```

## Microsoft Azure

### Prerequisites

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Set subscription
az account set --subscription "your-subscription-id"
```

### Terraform Configuration

#### Main Infrastructure (main.tf)

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }

  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "dealspherestate"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
}

variable "location" {
  description = "Azure region"
  type        = string
  default     = "East US"
}

variable "environment" {
  description = "Environment (staging/production)"
  type        = string
  default     = "production"
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "dealsphere-${var.environment}-rg"
  location = var.location

  tags = {
    Environment = var.environment
    Project     = "DealSphere"
  }
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "dealsphere-${var.environment}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  tags = {
    Environment = var.environment
  }
}

resource "azurerm_subnet" "public" {
  name                 = "public"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_subnet" "private" {
  name                 = "private"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]

  service_endpoints = ["Microsoft.Sql"]
}

# Network Security Groups
resource "azurerm_network_security_group" "public" {
  name                = "dealsphere-${var.environment}-public-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "HTTP"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "HTTPS"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "public" {
  subnet_id                 = azurerm_subnet.public.id
  network_security_group_id = azurerm_network_security_group.public.id
}
```

#### AKS Cluster (aks.tf)

```hcl
# AKS Cluster
resource "azurerm_kubernetes_cluster" "main" {
  name                = "dealsphere-${var.environment}-aks"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  dns_prefix          = "dealsphere-${var.environment}"

  default_node_pool {
    name       = "default"
    node_count = var.environment == "production" ? 2 : 1
    vm_size    = var.environment == "production" ? "Standard_D4s_v3" : "Standard_B2s"

    vnet_subnet_id = azurerm_subnet.private.id
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin = "azure"
    service_cidr   = "10.1.0.0/16"
    dns_service_ip = "10.1.0.10"
  }

  tags = {
    Environment = var.environment
  }
}

# Additional Node Pool for Production
resource "azurerm_kubernetes_cluster_node_pool" "production" {
  count = var.environment == "production" ? 1 : 0

  name                  = "production"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.main.id
  vm_size              = "Standard_D8s_v3"
  node_count           = 3
  vnet_subnet_id       = azurerm_subnet.private.id

  node_taints = ["workload=production:NoSchedule"]
  node_labels = {
    "workload" = "production"
  }

  tags = {
    Environment = var.environment
  }
}
```

#### PostgreSQL Database (postgresql.tf)

```hcl
# Random password for database
resource "random_password" "db_password" {
  length  = 32
  special = true
}

# PostgreSQL Server
resource "azurerm_postgresql_flexible_server" "main" {
  name                   = "dealsphere-${var.environment}-psql"
  resource_group_name    = azurerm_resource_group.main.name
  location               = azurerm_resource_group.main.location
  version                = "16"

  administrator_login    = "dealsphere"
  administrator_password = random_password.db_password.result

  storage_mb   = var.environment == "production" ? 102400 : 32768
  storage_tier = "P4"

  sku_name = var.environment == "production" ? "GP_Standard_D4s_v3" : "B_Standard_B1ms"
  zone     = "1"

  backup_retention_days        = var.environment == "production" ? 35 : 7
  point_in_time_restore_time_in_minutes = var.environment == "production" ? 10080 : 60

  high_availability {
    mode = var.environment == "production" ? "ZoneRedundant" : "Disabled"
  }

  tags = {
    Environment = var.environment
  }
}

# PostgreSQL Database
resource "azurerm_postgresql_flexible_server_database" "main" {
  name      = "dealsphere"
  server_id = azurerm_postgresql_flexible_server.main.id
  collation = "en_US.UTF8"
  charset   = "UTF8"
}

# Firewall rule for AKS subnet
resource "azurerm_postgresql_flexible_server_firewall_rule" "aks" {
  name             = "aks-subnet"
  server_id        = azurerm_postgresql_flexible_server.main.id
  start_ip_address = cidrhost(azurerm_subnet.private.address_prefixes[0], 0)
  end_ip_address   = cidrhost(azurerm_subnet.private.address_prefixes[0], -1)
}

# Key Vault for secrets
resource "azurerm_key_vault" "main" {
  name                       = "dealsphere-${var.environment}-kv-${random_id.kv_suffix.hex}"
  location                   = azurerm_resource_group.main.location
  resource_group_name        = azurerm_resource_group.main.name
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  sku_name                   = "standard"
  soft_delete_retention_days = 7

  access_policy {
    tenant_id = data.azurerm_client_config.current.tenant_id
    object_id = data.azurerm_client_config.current.object_id

    secret_permissions = [
      "Get",
      "List",
      "Set",
      "Delete",
      "Recover",
      "Backup",
      "Restore"
    ]
  }

  tags = {
    Environment = var.environment
  }
}

resource "random_id" "kv_suffix" {
  byte_length = 4
}

data "azurerm_client_config" "current" {}

# Store database password in Key Vault
resource "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  value        = random_password.db_password.result
  key_vault_id = azurerm_key_vault.main.id
}
```

## Digital Ocean (Simplified)

### Terraform Configuration

```hcl
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

variable "do_token" {
  description = "DigitalOcean API Token"
  type        = string
  sensitive   = true
}

# VPC
resource "digitalocean_vpc" "main" {
  name     = "dealsphere-${var.environment}-vpc"
  region   = "nyc3"
  ip_range = "10.10.0.0/16"
}

# Kubernetes Cluster
resource "digitalocean_kubernetes_cluster" "main" {
  name    = "dealsphere-${var.environment}"
  region  = "nyc3"
  version = "1.28.2-do.0"
  vpc_uuid = digitalocean_vpc.main.id

  node_pool {
    name       = "worker-pool"
    size       = var.environment == "production" ? "s-4vcpu-8gb" : "s-2vcpu-4gb"
    node_count = var.environment == "production" ? 3 : 2
    auto_scale = true
    min_nodes  = 1
    max_nodes  = var.environment == "production" ? 10 : 5
  }

  tags = ["dealsphere", var.environment]
}

# Database
resource "digitalocean_database_cluster" "main" {
  name       = "dealsphere-${var.environment}-db"
  engine     = "pg"
  version    = "16"
  size       = var.environment == "production" ? "db-s-4vcpu-8gb" : "db-s-1vcpu-2gb"
  region     = "nyc3"
  node_count = var.environment == "production" ? 2 : 1

  private_network_uuid = digitalocean_vpc.main.id

  tags = ["dealsphere", var.environment]
}

resource "digitalocean_database_db" "dealsphere" {
  cluster_id = digitalocean_database_cluster.main.id
  name       = "dealsphere"
}

resource "digitalocean_database_user" "dealsphere" {
  cluster_id = digitalocean_database_cluster.main.id
  name       = "dealsphere"
}

# Load Balancer
resource "digitalocean_loadbalancer" "main" {
  name     = "dealsphere-${var.environment}-lb"
  region   = "nyc3"
  vpc_uuid = digitalocean_vpc.main.id

  forwarding_rule {
    entry_protocol  = "http"
    entry_port      = 80
    target_protocol = "http"
    target_port     = 80
    redirect_http_to_https = true
  }

  forwarding_rule {
    entry_protocol  = "https"
    entry_port      = 443
    target_protocol = "http"
    target_port     = 80
    certificate_name = digitalocean_certificate.main.name
  }

  healthcheck {
    protocol = "http"
    port     = 80
    path     = "/health"
  }

  tags = ["dealsphere", var.environment]
}

# SSL Certificate
resource "digitalocean_certificate" "main" {
  name    = "dealsphere-${var.environment}-cert"
  type    = "lets_encrypt"
  domains = [var.domain_name]
}

# Container Registry
resource "digitalocean_container_registry" "main" {
  name                   = "dealsphere"
  subscription_tier_slug = "basic"
  region                 = "nyc3"
}
```

## CI/CD Integration

### GitHub Actions for Cloud Deployment

```yaml
# .github/workflows/deploy-cloud.yml
name: Deploy to Cloud

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

env:
  ENVIRONMENT: ${{ github.event.inputs.environment || 'staging' }}

jobs:
  deploy-infrastructure:
    name: Deploy Infrastructure
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Terraform Init
        run: terraform init
        working-directory: ./infrastructure/aws

      - name: Terraform Plan
        run: terraform plan -var="environment=${{ env.ENVIRONMENT }}"
        working-directory: ./infrastructure/aws

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve -var="environment=${{ env.ENVIRONMENT }}"
        working-directory: ./infrastructure/aws

  deploy-application:
    name: Deploy Application
    runs-on: ubuntu-latest
    needs: [deploy-infrastructure]

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker images
        run: |
          # Backend
          docker build -t $ECR_REGISTRY/dealsphere-backend:${{ github.sha }} ./backend
          docker push $ECR_REGISTRY/dealsphere-backend:${{ github.sha }}

          # Frontend
          docker build -t $ECR_REGISTRY/dealsphere-frontend:${{ github.sha }} ./frontend
          docker push $ECR_REGISTRY/dealsphere-frontend:${{ github.sha }}
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}

      - name: Deploy to ECS
        run: |
          # Update task definition with new image
          aws ecs update-service \
            --cluster dealsphere-${{ env.ENVIRONMENT }} \
            --service dealsphere-backend \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster dealsphere-${{ env.ENVIRONMENT }} \
            --services dealsphere-backend
```

## Monitoring and Observability

### Cloud-Native Monitoring

**AWS CloudWatch:**
- Application logs and metrics
- Custom dashboards and alarms
- Integration with ECS and RDS

**GCP Cloud Monitoring:**
- GKE cluster monitoring
- Application performance monitoring
- Log aggregation with Cloud Logging

**Azure Monitor:**
- AKS monitoring with Container Insights
- Application Insights for APM
- Log Analytics workspace

### Example CloudWatch Dashboard

```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/ECS", "CPUUtilization", "ServiceName", "dealsphere-backend"],
          [".", "MemoryUtilization", ".", "."]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-west-2",
        "title": "ECS Service Metrics"
      }
    },
    {
      "type": "log",
      "properties": {
        "query": "SOURCE '/ecs/dealsphere-production' | fields @timestamp, @message\n| filter @message like /ERROR/\n| sort @timestamp desc\n| limit 20",
        "region": "us-west-2",
        "title": "Recent Errors"
      }
    }
  ]
}
```

## Cost Optimization

### AWS Cost Optimization

**Reserved Instances:**
- RDS Reserved Instances for database
- EC2 Reserved Instances for predictable workloads
- ECS Fargate Savings Plans

**Resource Scheduling:**
- Auto Scaling for ECS services
- RDS instance scheduling for non-production
- CloudFront edge caching

**Storage Optimization:**
- S3 Intelligent Tiering
- EBS GP3 storage type
- Regular snapshot cleanup

### Multi-Cloud Cost Comparison

| Service | AWS | GCP | Azure | Digital Ocean |
|---------|-----|-----|--------|---------------|
| Compute (2 vCPU, 8GB) | $140/mo | $130/mo | $135/mo | $96/mo |
| Database (Small) | $85/mo | $75/mo | $80/mo | $60/mo |
| Load Balancer | $18/mo | $15/mo | $20/mo | $12/mo |
| Storage (100GB) | $10/mo | $10/mo | $12/mo | $10/mo |
| **Total (Est.)** | **$253/mo** | **$230/mo** | **$247/mo** | **$178/mo** |

## Disaster Recovery

### Multi-Region Setup

**AWS Multi-Region:**
```hcl
# Primary region: us-west-2
# DR region: us-east-1

resource "aws_s3_bucket_replication_configuration" "main" {
  bucket = aws_s3_bucket.main.id
  role   = aws_iam_role.replication.arn

  rule {
    id     = "replicate-to-dr"
    status = "Enabled"

    destination {
      bucket = aws_s3_bucket.dr.arn
    }
  }
}

# RDS Cross-Region Automated Backups
resource "aws_db_instance_automated_backups_replication" "main" {
  source_db_instance_arn = aws_db_instance.main.arn
  kms_key_id            = aws_kms_key.dr.arn
}
```

### Backup and Recovery Procedures

1. **Database Backups**: Automated daily backups with cross-region replication
2. **Application State**: Stateless applications with external state storage
3. **Configuration**: Infrastructure as Code in version control
4. **Monitoring**: Health checks and automated failover procedures

## Security Best Practices

### Infrastructure Security

**Network Security:**
- VPC with private subnets for databases
- Security groups with least privilege access
- WAF for web application protection

**Encryption:**
- Encryption at rest for all storage
- TLS 1.2+ for all communication
- KMS/Key Vault for key management

**Access Control:**
- IAM roles with minimal permissions
- Service accounts for application access
- Multi-factor authentication for admin access

## Troubleshooting

### Common Cloud Issues

**ECS/EKS Task Failures:**
```bash
# Check task logs
aws ecs describe-tasks --cluster my-cluster --tasks task-id
kubectl logs -f deployment/my-app

# Check service events
aws ecs describe-services --cluster my-cluster --services my-service
kubectl describe deployment my-app
```

**Database Connection Issues:**
```bash
# Test connectivity
telnet db-endpoint 5432
nslookup db-endpoint

# Check security groups
aws ec2 describe-security-groups --group-ids sg-xxxxx
```

**SSL Certificate Issues:**
```bash
# Check certificate status
aws acm list-certificates
gcloud compute ssl-certificates list
az network application-gateway ssl-cert list
```

## Related Documentation

- [Production Deployment](production-deployment.md)
- [Environment Management](environment-management.md)
- [Security and Secrets Management](security-secrets-management.md)
- [Observability and Monitoring](observability-monitoring.md)