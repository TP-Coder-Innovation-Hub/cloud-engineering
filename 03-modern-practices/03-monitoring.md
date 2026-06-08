# Monitoring `[Senior]`

## The Three Pillars

```yaml
pillars:
  metrics:
    description: "Numeric measurements over time"
    examples: ["CPU 85%", "request latency 230ms", "error rate 2.3%"]
    tools: ["CloudWatch", "Prometheus", "Azure Monitor"]
    query: "What is happening right now?"

  logs:
    description: "Discrete events with timestamps"
    examples: ["User logged in", "Order failed validation", "Connection timeout"]
    tools: ["CloudWatch Logs", "ELK Stack", "Loki"]
    query: "Why did something happen?"

  traces:
    description: "Request journey across services"
    examples: ["Frontend → API → DB: 450ms total"]
    tools: ["X-Ray", "Jaeger", "Application Insights"]
    query: "Where is the bottleneck?"
```

## Metrics

Metrics are numbers sampled over time. They answer "what is the state of the system?"

```yaml
# Key application metrics (RED method for services)
metrics:
  rate:
    description: "Requests per second"
    alert_if: "Sudden drop (service is down) or spike (DDoS)"

  errors:
    description: "Failed requests per second"
    alert_if: "Error rate exceeds 1% of total requests"

  duration:
    description: "Request latency (p50, p95, p99)"
    alert_if: "p99 latency exceeds 2 seconds"

# Key system metrics (USE method for resources)
  utilization:
    description: "Resource usage as percentage"
    alert_if: "CPU > 80%, Memory > 90%, Disk > 85%"

  saturation:
    description: "Queue depth, thread pool exhaustion"
    alert_if: "Request queue depth > 100"

  errors:
    description: "System-level errors"
    alert_if: "Disk I/O errors, network retransmits"
```

```yaml
# Metric alert definition
alert:
  name: "high-error-rate"
  metric: "http_server_errors / http_server_requests"
  threshold: 0.01            # 1% error rate
  window: "5 minutes"
  action: "page-on-call"
  severity: "critical"
```

## Logs

Logs are events. Each log entry has a timestamp, a level, and a message. Centralize them.

```yaml
# Structured logging (always use structured logs)
log_entry:
  timestamp: "2024-01-15T10:30:00Z"
  level: "ERROR"
  service: "order-service"
  trace_id: "abc-123-def"
  message: "Order validation failed"
  context:
    order_id: "ORD-456"
    reason: "invalid_item_count"
    user_id: "USR-789"
```

```yaml
log_aggregation:
  why: "100 instances logging locally means checking 100 machines"
  how: "All instances ship logs to central service"
  query: "Search all logs for trace_id=abc-123-def across all services"
  retention: "Hot (7 days, fast query) → Warm (30 days) → Cold (1 year, slow query)"
```

## Traces

A trace follows a single request across multiple services. Each service records a span.

```
Trace: abc-123-def (total: 450ms)
│
├── Span: API Gateway (5ms)
│   └── Span: order-service (200ms)
│       ├── Span: validate-order (20ms)
│       └── Span: db.insert (175ms) ← bottleneck
│           └── Span: database-query (170ms)
│
└── Span: notification-service (240ms)
    └── Span: email.send (235ms)
```

```yaml
# What traces tell you that metrics cannot
problem: "p99 latency is 2 seconds"
metrics: "Show you the number. Cannot show you the cause."
traces: "Show that database-query takes 1.8 seconds of the 2 seconds."
action: "Add a database index. p99 drops to 200ms."
```

## Alerting on Symptoms, Not Causes

```yaml
# Bad alert: alert on causes
alert:
  name: "cpu-high"
  condition: "CPU > 80%"
  problem: "Maybe CPU is high because of a batch job. Maybe the service is fine."

# Good alert: alert on symptoms
alert:
  name: "high-error-rate"
  condition: "Error rate > 1% for 5 minutes"
  reason: "Users are experiencing failures. Investigate now."

# Best: alert on user impact
alert:
  name: "users-affected-by-errors"
  condition: "More than 100 unique users saw errors in last 5 minutes"
```

Alert on what the user experiences. Use metrics and traces to diagnose why.

## Dashboard Hierarchy

```yaml
dashboards:
  level_1_executive:
    shows: "Service health, error rates, user impact"
    audience: "Leadership"
    purpose: "Is the business running?"

  level_2_service:
    shows: "RED metrics per service, throughput, latency distributions"
    audience: "On-call engineers"
    purpose: "Which service is having problems?"

  level_3_resource:
    shows: "CPU, memory, disk, network per instance"
    audience: "Engineers debugging"
    purpose: "Why is this service slow?"
```

## Key Takeaway

Metrics tell you what. Logs tell you why. Traces tell you where. Alert on symptoms (user impact), diagnose with causes (resource metrics). Centralize logs. Instrument traces. Build dashboards in layers.
