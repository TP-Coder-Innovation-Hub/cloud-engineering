# Alternatives

## Landscape

| Tool | Language | Cloud | Approach |
|------|----------|-------|----------|
| Terraform | HCL (declarative) | Multi-cloud | Resource graph, plan/apply |
| Pulumi | Python, TS, Go, C# | Multi-cloud | Real code, imperative logic |
| CDK | TypeScript, Python, Java | AWS, (CDKTF multi-cloud) | Object-oriented constructs |
| Bicep | DSL (declarative) | Azure only | Compiles to ARM templates |
| ARM Templates | JSON (declarative) | Azure only | Raw API mapping |

## Pulumi

Use actual programming languages. Loops, conditionals, classes, tests, and package management are native.

```python
# Pulumi: Python
import pulumi_aws as aws

# A for loop, not count meta-argument
for i in range(3):
    aws.ec2.Instance(f"web-{i}",
        ami="ami-0c55b159cbfafe1f0",
        instance_type="t3.medium",
        tags={"Name": f"web-{i}", "Environment": "prod"})
```

```yaml
when_pulumi:
  - "Team prefers real languages over DSLs"
  - "Need complex logic (generate config from external API)"
  - "Want unit tests for infrastructure"
  - "Existing codebase patterns to reuse"

when_not_pulumi:
  - "Team already knows Terraform"
  - "Hiring for Terraform is easier (more candidates)"
  - "Vast Terraform module ecosystem is needed"
```

## AWS CDK (Cloud Development Kit)

Define AWS infrastructure in TypeScript, Python, Java, or C#. Synthesizes to CloudFormation.

```typescript
// CDK: TypeScript
import * as cdk from 'aws-cdk-lib';
import * as s3 from 'aws-cdk-lib/aws-s3';

export class MyAppStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    new s3.Bucket(this, 'Uploads', {
      versioned: true,
      encryption: s3.BucketEncryption.S3_MANAGED,
      removalPolicy: cdk.RemovalPolicy.RETAIN,
    });
  }
}
```

```yaml
cdk_variants:
  cdk_for_terraform:
    description: "CDKTF -- same concept, outputs Terraform configs"
    clouds: "Multi-cloud (via Terraform providers)"
    benefit: "Pulumi-like experience with Terraform backend"
  aws_cdk:
    description: "Original CDK, outputs CloudFormation"
    clouds: "AWS only"
    benefit: "Best AWS integration, Constructs ecosystem"
```

## Bicep (Azure)

A cleaner DSL that compiles to ARM templates. Microsoft's answer to HCL for Azure.

```bicep
// Bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'mystorageaccount${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
    minimumTlsVersion: 'TLS1_2'
  }
}
```

```yaml
when_bicep:
  - "Azure-only shop"
  - "Want Microsoft-supported tooling"
  - "ARM templates are unreadable and you want a cleaner syntax"
  - "Integrating with Azure DevOps pipelines"
```

## Comparison: Choosing a Tool

```yaml
decision_framework:
  multi_cloud:
    preferred: "Terraform (industry standard) or Pulumi (if team wants real code)"
  aws_only:
    preferred: "Terraform or CDK (best AWS integration)"
  azure_only:
    preferred: "Bicep or Terraform"
  team_has_no_iac_experience:
    preferred: "Terraform (most learning resources available)"
  team_wants_type_safety:
    preferred: "Pulumi (TypeScript/Python) or CDK (TypeScript)"
  complex_logic_needed:
    preferred: "Pulumi (full programming language)"
```

## Key Takeaway

Terraform is the safe default. Most widely adopted, most community modules, most job postings. Pulumi or CDK are strong choices when your team wants real programming languages. Bicep is the right call for Azure-only teams. The concepts transfer. Learn one deeply.
