# Serverless

## Functions-as-a-Service

Write a function. Configure a trigger. The cloud runs it when the trigger fires. You pay per invocation.

```yaml
# Serverless function definition
function:
  name: "process-order"
  runtime: "python 3.11"
  memory: 256               # MB
  timeout: 30               # seconds
  trigger:
    type: "http"
    path: "/orders"
    method: "POST"
  environment:
    DATABASE_URL: "${secrets.db_url}"
    LOG_LEVEL: "info"
```

```python
# Pseudocode: the function
def handler(event, context):
    order = parse(event["body"])
    validate(order)
    db.insert("orders", order)
    notify("warehouse", order)
    return {"statusCode": 200, "body": json.dumps({"id": order["id"]})}
```

## Event-Driven Architecture

Serverless excels when work is triggered by events, not by constant user traffic.

> 🖼️ **[IMAGE_PLACEHOLDER]** — serverless event-driven architecture API gateway S3 SQS triggers

```yaml
event_sources:
  http_request:
    description: "API Gateway routes HTTP to function"
    example: "POST /orders triggers process-order"

  file_upload:
    description: "Object created in bucket triggers function"
    example: "Image uploaded to S3 triggers resize function"

  database_change:
    description: "Record inserted/modified triggers function"
    example: "New row in Orders table triggers notification"

  message_queue:
    description: "Message arrives in queue triggers function"
    example: "SQS message triggers email-sender function"

  schedule:
    description: "Cron-based trigger"
    example: "Every day at 02:00 UTC triggers cleanup function"
```

```
Event Source          Function              Downstream
─────────────        ──────────            ───────────
API Gateway ────► process-order ────► Database
                     │
                     └───────► Notification Service

S3 Upload  ────► resize-image  ────► Processed Bucket
             ────► extract-metadata ──► Metadata DB

Cron (daily) ──► cleanup-expired ──► Database
```

## When Serverless

```yaml
good_fit:
  - description: "Short-lived tasks"
    reason: "Pay only for execution time. No idle cost."
    example: "Image processing, data transformation"

  - description: "Event-triggered work"
    reason: "Natural fit for event-driven architecture."
    example: "Process webhook, handle file upload"

  - description: "Variable, unpredictable traffic"
    reason: "Scales from 0 to thousands automatically."
    example: "API for a startup with unknown growth"

  - description: "Glue between services"
    reason: "Lightweight, no infrastructure to manage."
    example: "Transform SQS message and forward to SNS"
```

## When NOT Serverless

```yaml
bad_fit:
  - description: "Long-running processes"
    reason: "15-minute execution limit on most platforms."
    example: "Video encoding, large batch processing"
    alternative: "Containers or VMs"

  - description: "Steady, predictable high load"
    reason: "Per-invocation pricing exceeds reserved capacity."
    example: "API handling 10,000 RPS continuously"
    alternative: "Containers with reserved instances"

  - description: "Stateful applications"
    reason: "No persistent local storage between invocations."
    example: "WebSocket server, in-memory cache"
    alternative: "Containers or VMs"

  - description: "Latency-sensitive hot paths"
    reason: "Cold starts add 100ms-1s on first invocation."
    example: "High-frequency trading, real-time gaming"
    alternative: "Always-on containers"
```

## Cold Starts

A cold start occurs when the platform creates a new function instance. The runtime must be loaded, code initialized, and dependencies resolved before the function executes.

```yaml
cold_start_characteristics:
  python_nodejs:
    cold_start: "100-500ms"
    warm_invocation: "1-10ms"
  java_csharp:
    cold_start: "1-5 seconds"      # JVM/CLR startup overhead
    warm_invocation: "1-10ms"

mitigations:
  - "Keep functions small (fewer dependencies to load)"
  - "Use provisioned concurrency for critical paths"
  - "Choose interpreted runtimes (Python, Node) for latency-sensitive functions"
  - "Reuse warm instances with keep-alive pings"
```

## Cost Model

```yaml
billing_dimensions:
  - "Number of invocations"
  - "Execution duration (rounded to nearest ms or 100ms)"
  - "Memory allocation (choose 128MB - 10GB)"

cost_examples:
  low_traffic_api:
    requests: "1,000 per day"
    avg_duration: "200ms"
    memory: "256MB"
    monthly_cost: "~$0.20"

  high_traffic_api:
    requests: "10,000,000 per day"
    avg_duration: "50ms"
    memory: "512MB"
    monthly_cost: "~$1,000"
```

## Key Takeaway

Serverless is a tool for event-driven, variable-scale workloads. It is not a universal replacement for servers. Use it when the workload fits. Use containers or VMs when it does not. The best architecture uses each where appropriate.
