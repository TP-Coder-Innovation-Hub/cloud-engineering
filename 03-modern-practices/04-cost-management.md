# Cost Management `[Mid]`

## Cloud Bills Shock

The cloud billing model is "pay for what you use." Without controls, "what you use" grows silently.

```yaml
common_shock_scenarios:
  - "Developer forgot to shut down test environment. $2,000 over weekend."
  - "S3 bucket with 50TB of logs, never deleted. $1,150/month."
  - "Database instance sized for production load in dev. $800/month wasted."
  - "Data transfer between regions. $3,000/month you did not expect."
  - "No tagging. Cannot tell which team owns which resource."
```

## Tagging Strategy

Tags are metadata attached to resources. Without tags, you cannot attribute cost.

```yaml
# Mandatory tags for every resource
tags:
  environment:
    values: ["dev", "staging", "prod"]
    purpose: "Identify non-production resources for shutdown"
  team:
    values: ["backend", "frontend", "data", "platform"]
    purpose: "Cost attribution to teams"
  service:
    values: ["user-api", "order-service", "analytics"]
    purpose: "Cost per application"
  cost_center:
    values: ["CC-100", "CC-200"]
    purpose: "Finance chargeback"
  managed_by:
    values: ["terraform", "manual"]
    purpose: "Identify unmanaged resources"
```

Enforce tagging with IAM policies. Reject resource creation without mandatory tags.

```yaml
# Policy: deny resource creation without tags
policy:
  effect: "Deny"
  condition: "Missing any of [environment, team, service]"
  message: "Resource must have environment, team, and service tags"
```

## Budgets and Alerts

```yaml
# Budget definition
budget:
  name: "monthly-total"
  amount: "$10,000"
  period: "monthly"
  alerts:
    - threshold: 50         # percent
      action: "notify-finance-team"
    - threshold: 80
      action: "notify-engineering-lead"
    - threshold: 100
      action: "notify-vp-engineering"
      action: "page-on-call"

  budget_by_team:
    backend: "$4,000"
    frontend: "$2,000"
    data: "$3,000"
    platform: "$1,000"
```

## Rightsizing

Most cloud resources are over-provisioned. Find and shrink them.

```yaml
rightsizing_process:
  step_1_analyze:
    action: "Check CPU and memory utilization over 14 days"
    tool: "Cloud provider advisor (AWS Compute Optimizer, Azure Advisor)"

  step_2_identify:
    target: "Instances with < 20% average CPU utilization"
    exception: "Do not downsize databases during peak hours"

  step_3_resize:
    before: "m6i.2xlarge (8 vCPU, 32GB) at $0.38/hr"
    after: "m6i.large (2 vCPU, 8GB) at $0.096/hr"
    savings: "$210/month per instance"

  step_4_verify:
    action: "Monitor for 48 hours after resize"
    rollback: "Resize back up if latency increases"
```

## Reserved Capacity

Cloud providers discount committed usage. Trade flexibility for lower cost.

```yaml
# Pricing comparison (example rates)
on_demand:
  instance: "m6i.large"
  price: "$0.096/hour"
  annual: "$841"

reserved_1_year:
  commitment: "Use this instance type for 1 year"
  discount: "~30% off"
  annual: "$589"

reserved_3_year:
  commitment: "Use this instance type for 3 years"
  discount: "~55% off"
  annual: "$378"

savings_plans:
  description: "Commit to $X/hour of compute spend, flexible on instance type"
  benefit: "Same discount, more flexibility"
```

**Rule:** Reserve only for baseline steady-state load. Keep variable load on on-demand. Over-reserving wastes money when workloads change.

## FinOps Mindset

```yaml
finops_principles:
  - "Cost is a first-class engineering concern, not a finance afterthought"
  - "Engineers need visibility into what their code costs to run"
  - "Optimize continuously, not once a year"
  - "Tag everything. Attribute everything. Hold teams accountable."
  - "Architecture decisions have cost implications. Factor them in."
```

```yaml
# Cost review cadence
weekly:
  - "Check budget alerts"
  - "Review anomalous spending"
  - "Identify new untagged resources"

monthly:
  - "Full cost report by team and service"
  - "Rightsizing recommendations review"
  - "Reserved capacity utilization check"

quarterly:
  - "Architecture review for cost optimization"
  - "Reserved capacity planning for next quarter"
  - "Team cost accountability review"
```

## Key Takeaway

Tag everything. Set budgets with alerts. Rightsize continuously. Reserve for baseline load. Make cost visible to engineers. Review spending weekly. Treat cloud cost like any other engineering metric: measure, alert, optimize.
