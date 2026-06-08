# Well-Architected `[Senior]`

## The Five Pillars

Every architectural decision involves trade-offs. The Well-Architected Framework provides a structured way to evaluate them.

```yaml
pillars:
  operational_excellence:
    question: "Can you operate and evolve the system?"
    focus: "Automation, runbooks, observability, continuous improvement"

  security:
    question: "Can you protect data, systems, and assets?"
    focus: "IAM, encryption, detection, incident response"

  reliability:
    question: "Can the system recover from failures and scale?"
    focus: "Multi-AZ, disaster recovery, testing failures"

  performance_efficiency:
    question: "Can you use resources efficiently?"
    focus: "Right-sizing, caching, serverless, GPU where needed"

  cost_optimization:
    question: "Can you avoid unnecessary expense?"
    focus: "Rightsizing, reserved capacity, tagging, FinOps"
```

No pillar exists in isolation. Improving one often impacts others.

## Pillar 1: Operational Excellence

```yaml
practices:
  infrastructure_as_code:
    - "All infrastructure defined in code"
    - "Changes via CI/CD pipeline, never manual"
    - "Code-reviewed before application"

  observability:
    - "Metrics for every service"
    - "Centralized logs with structured format"
    - "Distributed tracing across services"
    - "Dashboards per service, not per instance"

  runbooks:
    - "Written procedures for every alert"
    - "Tested quarterly (not just written and forgotten)"
    - "Include rollback steps"

  continuous_improvement:
    - "Post-incident reviews after every outage"
    - "Action items tracked and completed"
    - "Regular game days (deliberately break things)"
```

## Pillar 2: Security

```yaml
practices:
  identity:
    - "Principle of least privilege for every role"
    - "MFA for all human access"
    - "Temporary credentials, not long-lived keys"
    - "Regular permission audits"

  detection:
    - "Log all API calls (CloudTrail, Activity Log)"
    - "Alert on anomalous access patterns"
    - "Vulnerability scanning on schedule"
    - "Config drift detection"

  protection:
    - "Network segmentation (public/app/data tiers)"
    - "Encryption at rest and transit"
    - "WAF for public-facing endpoints"
    - "DDoS protection (Shield, Cloudflare)"

  incident_response:
    - "Documented response procedures"
    - "On-call rotation with clear escalation"
    - "Automated containment for known threats"
```

## Pillar 3: Reliability

```yaml
practices:
  foundations:
    - "Multi-AZ for all production workloads"
    - "Defined RPO and RTO per workload"
    - "Automatic failover for databases"
    - "Circuit breakers for downstream calls"

  change_management:
    - "Small, frequent deployments (lower risk than large batch)"
    - "Canary or blue-green deployment strategies"
    - "Automated rollback on health check failure"
    - "Feature flags to limit blast radius"

  failure_management:
    - "Chaos engineering (intentionally inject failures)"
    - "Disaster recovery drills quarterly"
    - "Backup restoration tested monthly"
    - "Dependency failure mode analysis"
```

## Pillar 4: Performance Efficiency

```yaml
practices:
  resource_selection:
    - "Benchmark before choosing instance types"
    - "Use GPU instances only for GPU workloads"
    - "Consider Graviton/ARM for compatible workloads (better price-performance)"
    - "Evaluate serverless for variable workloads"

  caching:
    - "CDN for static assets"
    - "Application cache (Redis/Memcached) for repeated queries"
    - "Browser caching headers for client-side resources"

  optimization:
    - "Database query optimization before scaling up"
    - "Connection pooling for database connections"
    - "Async processing for non-blocking operations"
    - "Right-size based on actual utilization, not guesses"
```

## Pillar 5: Cost Optimization

```yaml
practices:
  visibility:
    - "Tag all resources"
    - "Budgets with alerts per team"
    - "Monthly cost review meetings"

  efficiency:
    - "Rightsize instances based on utilization data"
    - "Use spot/preemptible instances for fault-tolerant workloads"
    - "Reserved capacity for steady-state baseline"
    - "Auto-stop non-production environments outside business hours"

  trade_offs:
    - "Serverless for sporadic workloads (pay per use)"
    - "Reserved instances for steady workloads (committed discount)"
    - "Spot instances for batch processing (cheapest, interruptible)"
```

## Trade-offs

```yaml
examples:
  reliability_vs_cost:
    decision: "Multi-region active-active"
    reliability: "Maximum. Survives region failure."
    cost: "2-3x. Running everything in multiple regions."
    when: "Financial services, healthcare, large-scale SaaS"

  performance_vs_cost:
    decision: "Over-provision database vs rightsize"
    performance: "Headroom for traffic spikes."
    cost: "Paying for unused capacity."
    when: "Over-provision if outage cost exceeds capacity cost"

  security_vs_velocity:
    decision: "Security review gate in CI/CD"
    security: "Catches vulnerabilities before deployment."
    velocity: "Adds time to every deployment."
    when: "Always. Security cannot be optional. Automate the review."
```

## Key Takeaway

Every architecture decision involves trade-offs across these five pillars. There is no perfect architecture. There are informed trade-offs aligned with business requirements. Review your architecture against these pillars regularly. Fix the gaps that matter most first.
