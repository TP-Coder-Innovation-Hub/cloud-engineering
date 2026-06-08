# Why IaC

## ClickOps vs Code

> 🖼️ **[IMAGE_PLACEHOLDER]** — clickOps manual console vs infrastructure as code Git workflow

```yaml
clickops:
  how: "Log into console, click through UI, create resources"
  time_to_create: "5 minutes"
  time_to_recreate: "Which settings did I use? Let me check..."
  audit_trail: "None"
  reproducibility: "Hope you remember every click"
  rollback: "Manually undo each change"
  peer_review: "Screenshot of the console?"

infrastructure_as_code:
  how: "Write code that describes resources, run it to create them"
  time_to_create: "5 minutes (first time), 30 seconds (repeat)"
  time_to_recreate: "Run the same code"
  audit_trail: "Git history, every change tracked"
  reproducibility: "Identical, every time"
  rollback: "git revert, re-apply"
  peer_review: "Pull request with full diff"
```

## The Problem With Manual Configuration

```yaml
# Day 1: "I'll just create this one bucket quickly"
- Logged into console
- Clicked create bucket
- Named it "my-bucket"
- Left all defaults
- Did not enable versioning
- Did not enable encryption
- Did not block public access

# Day 90: "We need a staging environment identical to production"
- Nobody remembers the production settings
- Nobody knows who made changes since Day 1
- Staging looks different from production
- Bugs appear only in staging (or only in production)

# Day 180: "Someone deleted the production database"
- No record of what the configuration was
- Recreating it takes hours of guesswork
- The configuration drifts further
```

## What IaC Solves

**Reproducibility.** The same code produces the same infrastructure. Dev, staging, production are identical.

**Version control.** Every infrastructure change is a commit. Who changed what, when, and why. You can diff infrastructure like code.

**Auditability.** Compliance requirements (SOC 2, HIPAA) need proof of what exists and what changed. Git log provides this.

**Automation.** Infrastructure changes go through the same CI/CD pipeline as application code. Test, review, apply.

**Self-documentation.** The code IS the documentation. No stale wiki pages describing what someone thinks the infrastructure looks like.

## The Mental Shift

```yaml
old_way:
  thought: "I need a server"
  action: "Click through console"
  result: "One server exists, only I know its config"

new_way:
  thought: "I need a server"
  action: "Write resource block in Terraform"
  action: "Commit, create PR, get review"
  action: "Merge, CI applies automatically"
  result: "Server exists, config is documented, reproducible, auditable"
```

## IaC Is Not Slower

The first resource takes longer with IaC. The second resource is faster. The hundredth resource takes the same time as the first. Manual configuration does not scale. Clicking through a console to create 50 identical environments is a waste of human time.

```yaml
# One server or one hundred, the code is the same
resource "compute_instance" "web" {
  count        = var.instance_count    # 1 or 100
  name         = "web-${count.index}"
  machine_type = "e2-medium"
  zone         = "us-east1-a"
}
```

## Key Takeaway

If you create it in a console, it does not exist in a reproducible way. If it is not reproducible, you cannot reliably recreate it. If you cannot recreate it, you are one mistake away from an outage with no recovery plan. Write code. Review it. Apply it.
