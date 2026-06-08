# Cloud Providers `[Entry]`

## The Big Three

| Provider | Market Share (approx) | Strength |
|----------|----------------------|----------|
| AWS | ~31% | Broadest service catalog, largest ecosystem |
| Azure | ~25% | Enterprise integration, Microsoft stack |
| GCP | ~11% | Data/analytics, Kubernetes (they built it) |

## When Each Excels

**AWS** -- Default choice for most workloads. Most services, most community content, most job postings. If you have no strong opinion, start here.

**Azure** -- Enterprises already running Active Directory, Office 365, or .NET workloads. Hybrid cloud scenarios with on-prem Windows Server. Government workloads (Azure Government).

**GCP** -- Data-heavy workloads (BigQuery is best-in-class). ML/AI workloads (TPUs, Vertex AI). Organizations running Kubernetes at scale (GKE is the managed K8s benchmark).

## Mapping Core Services

| Concept | AWS | Azure | GCP |
|---------|-----|-------|-----|
| VMs | EC2 | Virtual Machines | Compute Engine |
| Object Storage | S3 | Blob Storage | Cloud Storage |
| Managed DB | RDS | Azure SQL | Cloud SQL |
| Functions | Lambda | Azure Functions | Cloud Functions |
| Kubernetes | EKS | AKS | GKE |
| IAM | IAM | Entra ID + RBAC | IAM |
| VPC | VPC | VNet | VPC |

The concepts transfer. Learn one provider deeply, the others follow.

## Multi-Cloud Reality

Most enterprises end up multi-cloud through acquisition, not strategy. A company uses AWS, acquires a company on Azure, and now operates both.

Multi-cloud by design is expensive:
- No shared networking between providers
- Each provider has proprietary services
- Team expertise fragments across platforms
- Tooling complexity doubles

A pragmatic approach: pick a primary provider. Use another only when a specific capability justifies the overhead.

## Lock-In Concerns

Lock-in exists on a spectrum:

- **Low lock-in:** VMs running standard Linux, containers with open-source images
- **Medium lock-in:** Managed databases (proprietary APIs but standard SQL), serverless functions
- **High lock-in:** Proprietary services (DynamoDB, BigQuery, Cosmos DB), provider-specific orchestration

Mitigation strategies:
- Use containers over provider-specific runtimes
- Use standard protocols (SQL, HTTP, AMQP)
- Abstract provider APIs behind interfaces in your code
- Terraform (or Pulumi) to keep infrastructure portable-in-theory

Accept some lock-in. The cost of avoiding all lock-in exceeds the cost of migration.
