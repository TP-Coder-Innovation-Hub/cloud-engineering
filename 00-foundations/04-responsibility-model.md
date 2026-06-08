# Responsibility Model

## Shared Responsibility

The cloud provider secures the infrastructure. You secure what you put on it. The line between "their job" and "your job" depends on the service model.

> 🖼️ **[IMAGE_PLACEHOLDER]** — shared responsibility model provider vs customer SaaS PaaS IaaS

```
Responsibility         SaaS    PaaS    IaaS    On-Prem
─────────────────────  ──────  ──────  ──────  ────────
Physical security      Them    Them    Them    You
Network infrastructure Them    Them    Them    You
Host OS               Them    Them    Them    You
Runtime/Platform      Them    Them    You      You
Application code       You     You     You      You
Data                   You     You     You      You
Identity/IAM           You     You     You      You
```

## What the Provider Manages

The physical layer. Data center security, hardware lifecycle, network backbone, power, cooling. The hypervisor that isolates your VM from someone else's VM. The physical disks that store your data. The fiber cables between regions.

You will never touch a server. You will never patch a hypervisor. You will never replace a failed disk.

## What You Manage

**Always your responsibility, regardless of model:**

```yaml
your_responsibilities:
  - identity_and_access:
      description: "Who can access what"
      common_failure: "S3 bucket left public"
  - data_classification:
      description: "What data is sensitive"
      common_failure: "PII stored unencrypted"
  - client_side_encryption:
      description: "Encrypting data before sending"
      common_failure: "HTTP instead of HTTPS"
  - application_logic:
      description: "Your code, your bugs"
      common_failure: "SQL injection vulnerability"
```

## The Dangerous Middle Ground

Managed services blur the line:

```yaml
# RDS (managed database)
provider:
  - OS patching
  - Database engine patching
  - Backup automation
  - Hardware failures
you:
  - Schema design
  - Query performance
  - Access control (who can connect)
  - Encryption settings (enable at-rest encryption)

# S3 (object storage)
provider:
  - Durability (11 nines)
  - Server-side encryption option
  - Infrastructure
you:
  - Bucket policies (who can read/write)
  - Versioning (enable it)
  - Public access blocks (enable them)
```

## Security Of the Cloud vs Security In the Cloud

**Of the cloud:** Provider's job. Physical security, compute isolation, network infrastructure, DDoS protection at the edge.

**In the cloud:** Your job. IAM policies, security groups, encryption configuration, patching your application, managing secrets.

## The Breach Pattern

Most cloud breaches follow the same pattern:

```yaml
breach_pattern:
  step_1: "Misconfigured IAM policy grants public access"
  step_2: "Sensitive data in an S3 bucket accessible to the internet"
  step_3: "Attacker discovers the open bucket via scanning"
  step_4: "Data exfiltration"
  root_cause: "Customer misconfiguration, not provider failure"
```

The provider did not fail. The customer left the door open. Understanding where the responsibility line sits is not academic. It is the difference between a secure system and a headline.
