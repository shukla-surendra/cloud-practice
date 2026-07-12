# Terraform Complete Reference (Part 2)

> Deep dive into Terraform State, Backends, Dependency Graph, and Enterprise Best Practices.

# 11. What is Terraform State?

Terraform state is Terraform's record of the infrastructure addresses, resource identities, and metadata for what it manages.

It maps your Terraform configuration to real cloud resources.

Example:

```text
Terraform Code
      |
      v
terraform.tfstate
      |
      v
AWS Resources
```

Terraform uses the state to know:
- What already exists
- What needs to be created
- What needs updating
- What needs deleting

---

# 12. Why is State Required?

Without state Terraform would have to rediscover every resource every time.

State stores:

- Resource IDs
- Metadata
- Dependencies
- Outputs

Never edit the state file manually.

---

# 13. Local vs Remote State

## Local State

```text
terraform.tfstate
```

Pros:
- Simple
- Good for learning

Cons:
- Not shared
- Easy to lose
- Poor fit for team collaboration

## Remote State (Enterprise)

Typical AWS backend:

```text
Terraform
   |
S3 Bucket
   |
S3 Lockfile
```

Benefits:
- Shared
- Versioned
- Encrypted
- Locked
- Recoverable

---

# 14. Backend Configuration

Example:

```hcl
terraform {
  backend "s3" {
    bucket       = "company-tf-state"
    key          = "payments/dev/terraform.tfstate"
    region       = "ap-south-1"
    use_lockfile = true
    encrypt      = true
  }
}
```

Older configurations may still use `dynamodb_table` for locking, but HashiCorp has deprecated DynamoDB-based locking for the S3 backend in favor of native S3 lockfiles.

---

# 15. State Locking

Problem:

Engineer A and Engineer B both run:

```bash
terraform apply
```

Without locking, both modify the same state.

Result:
- Corruption
- Failed deployments

With backend locking enabled:

```text
Engineer A
     |
     |-- Lock Acquired
     |
Terraform Apply
     |
Release Lock

Engineer B waits...
```

In modern S3 backend setups, this is typically handled with `use_lockfile = true`.

---

# 16. One State or Many?

❌ One giant state

```text
company.tfstate
```

Problems:
- Slow
- Large blast radius
- Merge conflicts

✅ Better

```text
payments/dev.tfstate
payments/prod.tfstate

crm/dev.tfstate
crm/prod.tfstate

network/prod.tfstate
```

Rule:
> One independently deployable project/environment should have its own state.

---

# 17. Common State Commands

Initialize backend:

```bash
terraform init
```

List resources:

```bash
terraform state list
```

Show resource:

```bash
terraform state show aws_instance.web
```

Move state:

```bash
terraform state mv OLD NEW
```

Remove from state:

```bash
terraform state rm RESOURCE
```

Import existing resource:

```bash
terraform import aws_s3_bucket.logs my-bucket
```

Current Terraform versions also support `import` blocks in configuration. For team workflows, declarative imports are easier to review than one-off CLI imports.

Example:

```hcl
import {
  to = aws_s3_bucket.logs
  id = "my-bucket"
}
```

---

# 18. Dependency Graph

Terraform builds a graph before deployment.

```text
VPC
 |
Subnets
 |
EKS
 |
Application
```

Terraform automatically deploys in dependency order.

Generate the graph:

```bash
terraform graph
```

---

# 19. Day-to-Day Workflow

```text
terraform init
   |
terraform fmt
   |
terraform validate
   |
terraform plan
   |
Review
   |
terraform apply
```

In mature teams, `plan` and `apply` usually happen in CI/CD for shared environments rather than directly on a developer laptop.

---

# 20. Enterprise Best Practices

- Store state remotely.
- Enable S3 versioning.
- Enable bucket encryption.
- Enable backend locking.
- Keep one state per application/environment.
- Never store state in Git.
- Never manually edit state.
- Review every plan before apply.
- Pin Terraform and provider versions.
- Separate platform, networking, and application repositories.
- Use reusable modules.
- Protect production with approvals.
- Tag every resource.
- Document every module.
- Automate through CI/CD.

---

# 21. Common Mistakes

❌ One state for the whole company

❌ Local state in production

❌ Sharing tfvars between unrelated projects

❌ No locking

❌ Running apply directly on production

❌ Editing terraform.tfstate manually

❌ Hardcoded credentials

---

# Summary

Terraform's power comes from its state.

Understand these principles:

- State is Terraform's memory.
- Backends make collaboration safe.
- Locking prevents corruption.
- Small isolated states scale better.
- Dependency graphs determine execution order.
- Enterprise teams automate everything through CI/CD and remote state.
