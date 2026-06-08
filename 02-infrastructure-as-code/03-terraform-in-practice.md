# Terraform in Practice `[Senior]`

## Modules

A module is a reusable Terraform package. Put related resources together, parameterize them, use them across environments.

```hcl
# Directory structure
modules/
  web-app/
    main.tf       # resource definitions
    variables.tf  # input variables
    outputs.tf    # exported values
    versions.tf   # provider version constraints

environments/
  dev/
    main.tf       # calls module with dev values
  prod/
    main.tf       # calls module with prod values
```

```hcl
# modules/web-app/main.tf
variable "environment" { type = string }
variable "instance_count" { type = number }
variable "instance_type" { type = string }

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  tags = {
    Environment = var.environment
    Name        = "web-${var.environment}-${count.index}"
  }
}

output "instance_ids" {
  value = aws_instance.web[*].id
}
```

```hcl
# environments/prod/main.tf
module "web_app" {
  source         = "../../modules/web-app"
  environment    = "production"
  instance_count = 3
  instance_type  = "m6i.large"
}
```

## Variables and tfvars

```hcl
# variables.tf -- declare variables with types and defaults
variable "region" {
  type        = string
  default     = "us-east-1"
  description = "AWS region"
}

variable "allowed_cidrs" {
  type        = list(string)
  default     = ["10.0.0.0/8"]
}
```

```hcl
# environments/prod.tfvars -- set values per environment
region       = "us-east-1"
instance_type = "m6i.large"
allowed_cidrs = ["10.0.0.0/8", "172.16.0.0/12"]
```

```bash
terraform apply -var-file=environments/prod.tfvars
```

## Workspaces

Workspaces let you use the same configuration for multiple environments with separate state files.

```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch
terraform workspace select prod

# Use workspace name in resources
terraform workspace
```

```hcl
# Conditional sizing based on workspace
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "m6i.large" : "t3.medium"
}
```

**Limitation:** Workspaces share the same code. For significant environment differences, use directory-per-environment with modules. Workspaces work best for identical infrastructure at different scales.

## Remote State

State must be shared when a team works on the same infrastructure.

```hcl
# Backend configuration (in versions.tf or backend.tf)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"   # state locking
    encrypt        = true
  }
}
```

```yaml
# S3 backend setup
bucket:
  name: "my-terraform-state"
  versioning: enabled    # state history
  encryption: enabled

dynamodb_table:
  name: "terraform-locks"
  purpose: "Prevent concurrent apply operations"
```

**Without state locking:** Two engineers run `terraform apply` simultaneously. Both read the same state. Both write changes. One overwrites the other. Resources go orphaned.

## Organizing Real Projects

```yaml
# Recommended structure for a real project
project_root/
  modules/              # reusable components
    networking/
    compute/
    database/
  environments/
    dev/
      main.tf
      tfvars
    staging/
      main.tf
      tfvars
    prod/
      main.tf
      tfvars
  global/               # cross-environment resources
    iam/
    dns/
```

```yaml
principles:
  - "One state file per environment, per logical component"
  - "Small state files (networking separate from compute)"
  - "Modules for anything used more than once"
  - "Variables for anything that changes between environments"
  - "Outputs to pass data between components"
```

## Key Takeaway

Modules for reuse. Variables for flexibility. Remote state for teams. Separate environments with separate state. Keep state files small. Structure your Terraform like you structure your code.
