# Capstone: Deploy a 3-Tier Application `[Senior]`

## Objective

Design and implement a production-ready 3-tier application with high availability, monitoring, and infrastructure as code.

## Architecture

```
                     Internet
                        │
                        ▼
                  ┌───────────┐
                  │ Cloud DNS │  api.example.com
                  └─────┬─────┘
                        │
                        ▼
                  ┌───────────┐
                  │    CDN    │  Static assets
                  └─────┬─────┘
                        │
                        ▼
              ┌──────────────────┐
              │  Load Balancer   │  (HTTPS, multi-AZ)
              └────────┬─────────┘
                       │
           ┌───────────┼───────────┐
           ▼           ▼           ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │  Web/App │ │  Web/App │ │  Web/App │  Tier 1: Application
    │  AZ-1a   │ │  AZ-1b   │ │  AZ-1c   │  (Containers or VMs)
    └────┬─────┘ └────┬─────┘ └────┬─────┘
         │             │             │
         └─────────────┼─────────────┘
                       │
              ┌────────▼────────┐
              │   Cache Layer   │  (Redis, sub-ms reads)
              └────────┬────────┘
                       │
           ┌───────────┼───────────┐
           ▼           ▼           ▼
    ┌──────────┐                ┌──────────┐
    │ DB Primary│ ──replication─ │DB Replica │  Tier 2: Database
    │  AZ-1a   │                │  AZ-1b   │  (Managed SQL)
    └──────────┘                └──────────┘
                       │
                       ▼
              ┌────────────────┐
              │  Object Store  │  Tier 3: Storage
              │  (S3 / Blob)   │  (User uploads, static)
              └────────────────┘
```

## Requirements

### High Availability
- Multi-AZ deployment (minimum 2 AZs)
- Automatic database failover
- Load balancer health checks
- Application auto-scaling (min 2, max 6 instances)

### Infrastructure as Code
- All resources defined in Terraform
- Separate modules for networking, compute, database, storage
- Remote state with locking
- Variable files for dev and prod environments

### Monitoring
- Metrics: CPU, memory, request latency, error rate
- Logs: centralized, structured, queryable
- Traces: distributed tracing across all tiers
- Alerts: error rate > 1%, p99 latency > 2s, CPU > 80%

### Security
- HTTPS only (TLS 1.2+)
- Encryption at rest for database and object store
- Secrets in secrets manager
- Security groups: LB -> App -> DB (no skip tiers)
- IAM roles for service-to-service access

## Implementation Steps

```yaml
step_1_networking:
  resources:
    - "VPC with CIDR 10.0.0.0/16"
    - "2 public subnets (for LB) in different AZs"
    - "2 private subnets (for app) in different AZs"
    - "2 private subnets (for DB) in different AZs"
    - "Internet Gateway"
    - "NAT Gateway in each public subnet"
    - "Route tables for public and private subnets"
    - "Security groups for LB, app, and DB tiers"
  verify: "App instances can reach internet via NAT. DB subnet has no internet route."

step_2_storage:
  resources:
    - "S3 bucket for uploads (versioning, encryption, public access blocked)"
    - "S3 bucket for static assets (public read via CloudFront OAI)"
    - "CloudFront distribution for static assets"
  verify: "Upload file via app. Static assets served via CDN."

step_3_database:
  resources:
    - "Managed PostgreSQL instance (multi-AZ)"
    - "Read replica in second AZ"
    - "Automated backups (7 day retention)"
    - "Encryption at rest with KMS"
  verify: "Connect from app subnet. Failover works. Backup restores."

step_4_compute:
  resources:
    - "Container image in ECR (or VM AMI)"
    - "Auto-scaling group or ECS service (min 2, max 6)"
    - "Application Load Balancer (HTTPS, health checks)"
    - "IAM role for app (S3 read/write, Secrets Manager read)"
  verify: "App serves traffic via LB. Scaling triggers work. Health checks remove unhealthy instances."

step_5_monitoring:
  resources:
    - "CloudWatch metrics for all resources"
    - "Log groups for application logs"
    - "X-Ray tracing enabled"
    - "Dashboard with RED metrics"
    - "Alerts for error rate, latency, CPU"
  verify: "Generate errors. Verify alert fires. Check trace in console."

step_6_dns:
  resources:
    - "Route 53 hosted zone for example.com"
    - "A record for api.example.com -> LB alias"
    - "CNAME for cdn.example.com -> CloudFront domain"
    - "ACM certificate for *.example.com"
  verify: "curl https://api.example.com/health returns 200."
```

## Deliverables

```yaml
deliverables:
  terraform:
    - "modules/networking/"
    - "modules/compute/"
    - "modules/database/"
    - "modules/storage/"
    - "modules/monitoring/"
    - "environments/dev/"
    - "environments/prod/"
    - "remote state configuration"

  documentation:
    - "Architecture diagram"
    - "Runbook for common operations (scale up, failover, restore)"
    - "Cost estimate for dev and prod environments"

  validation:
    - "terraform plan shows no changes after apply (no drift)"
    - "Application accessible via HTTPS"
    - "Simulate AZ failure: app continues serving"
    - "Generate load: auto-scaling triggers"
    - "Inject error: alert fires within 5 minutes"
```

## Success Criteria

- Application is accessible via `https://api.example.com/health` returning 200
- All infrastructure is defined in Terraform with no manual steps
- Killing one AZ does not cause user-visible downtime
- Error rate alert fires within 5 minutes of injected failures
- All data is encrypted at rest and in transit
- Monthly cost estimate is documented and within budget
