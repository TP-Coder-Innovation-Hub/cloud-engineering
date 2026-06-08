# Security Basics

## Encryption

### At Rest

Data stored on disk is encrypted. If someone physically removes the disk, they get encrypted data.

```yaml
at_rest_encryption:
  storage:
    - "S3: server-side encryption (SSE-S3 or SSE-KMS)"
    - "EBS: encrypt volumes on creation (cannot add later)"
    - "RDS: enable encryption at creation (cannot add later)"
    - "EFS: enable encryption at creation"

  key_management:
    service: "KMS (Key Management Service)"
    options:
      - "Provider-managed keys (simplest, no key management)"
      - "Customer-managed keys (more control, audit key usage)"
      - "Customer-provided keys (you manage the key material)"

  rule: "Enable encryption by default. There is no performance penalty. No reason not to."
```

### In Transit

Data moving between client and server, or between services, is encrypted. Prevents eavesdropping.

```yaml
in_transit_encryption:
  external:
    - "HTTPS (TLS 1.2+) for all client-facing endpoints"
    - "HSTS headers to enforce HTTPS"
    - "Certificate management via ACM or Let's Encrypt"

  internal:
    - "TLS between microservices (mTLS for zero-trust)"
    - "TLS to database connections"
    - "VPN or TLS for VPC peering cross-region"

  rule: "No HTTP. Ever. All traffic must be TLS."
```

## Secrets Management

```yaml
# What is a secret
secrets:
  - "Database passwords"
  - "API keys"
  - "TLS private keys"
  - "OAuth client secrets"
  - "Encryption keys"

# Where NOT to put secrets
bad_locations:
  - "Source code (committed to Git)"
  - "Environment variables in CI config (visible in logs)"
  - "Plain text files on servers"
  - "Slack messages, emails, wikis"
  - "Docker image layers"
```

```yaml
# Where to put secrets
secrets_manager:
  service: "AWS Secrets Manager / Azure Key Vault / GCP Secret Manager"
  features:
    - "Encrypted storage"
    - "Automatic rotation"
    - "Access auditing"
    - "Fine-grained IAM policies"
    - "Injection at runtime (never in code)"

  usage_pattern:
    step_1: "Store secret in secrets manager"
    step_2: "Grant application role access to that specific secret"
    step_3: "Application fetches secret at startup"
    step_4: "Secret lives in memory, never on disk"
    step_5: "Rotate secret on schedule, update application"
```

## Network Isolation

```yaml
# Layered network security
layers:
  public_tier:
    resources: ["Load balancer", "CDN"]
    access: "Internet"
    security_group: "Allow 443 from anywhere"

  application_tier:
    resources: ["Web servers", "API servers"]
    access: "Load balancer only"
    security_group: "Allow 8080 from load balancer SG"

  data_tier:
    resources: ["Databases", "Cache", "Message queue"]
    access: "Application tier only"
    security_group: "Allow 5432 from application SG"

  management_tier:
    resources: ["Bastion host", "CI/CD runners"]
    access: "Corporate VPN only"
    security_group: "Allow 22 from VPN CIDR"
```

**Rule:** No resource in a lower tier can be reached directly from a higher tier. Traffic flows one direction through security groups.

## Compliance

```yaml
common_frameworks:
  SOC_2:
    scope: "Service organization controls"
    requirements: ["Access control", "Encryption", "Monitoring", "Incident response"]

  HIPAA:
    scope: "Healthcare data (US)"
    requirements: ["Encryption at rest and transit", "Access audit logs", "BAA with provider"]

  GDPR:
    scope: "Personal data of EU residents"
    requirements: ["Data residency", "Right to deletion", "Breach notification within 72 hours"]

  PCI_DSS:
    scope: "Payment card data"
    requirements: ["Network segmentation", "Encryption", "Access logging", "Vulnerability scanning"]
```

Compliance is not a one-time checklist. It is continuous:
- Log all access to sensitive resources
- Review IAM permissions quarterly
- Scan for misconfigurations continuously
- Encrypt everything
- Audit and rotate secrets

## Practical Security Checklist

```yaml
day_1:
  - "Enable encryption on all storage"
  - "Block all public S3 access at account level"
  - "Enable CloudTrail / Activity Logs (audit trail)"
  - "Require MFA for all human users"
  - "Create separate accounts/subscriptions per environment"

ongoing:
  - "Scan for public resources weekly"
  - "Rotate secrets on schedule"
  - "Patch OS and runtime vulnerabilities"
  - "Review IAM policies for over-provisioning"
  - "Test incident response runbook quarterly"
```

## Key Takeaway

Encrypt at rest and in transit. Store secrets in a secrets manager, never in code. Isolate network tiers with security groups. Automate compliance checks. Security is continuous, not a milestone.
