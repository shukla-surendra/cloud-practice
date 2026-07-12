# Terraform Complete Reference (Part 3)

> Enterprise project organization, modules, environments, and team collaboration.

# 22. Repository Organization

There is no single "correct" repository structure. Choose one based on team size.

## Option 1 - Monorepo

```text
company-infra/
├── networking/
├── security/
├── shared-services/
├── payments/
├── hr/
└── analytics/
```

**Pros**
- One place for everything
- Easier global refactoring

**Cons**
- Merge conflicts
- Larger CI/CD pipelines
- Harder ownership

Best for: Small teams.

---

## Option 2 - Repository Per Project (Recommended)

```text
payments-infra/
customer-portal-infra/
analytics-infra/
```

**Pros**
- Independent deployments
- Clear ownership
- Smaller pipelines
- Easier permissions

**Cons**
- More repositories

Best for: Medium and large organizations.

---

# 23. Typical Enterprise Layout

```text
terraform-modules/
│
├── networking/
├── compute/
├── storage/
├── database/
└── security/

payments-infra/
customer-portal-infra/
analytics-infra/
```

Platform teams own reusable modules.

Application teams own application repositories.

---

# 24. Root Module vs Child Module

Every deployment directory has **one Root Module**.

```text
payments/
├── main.tf
├── variables.tf
└── outputs.tf
```

The root module calls child modules.

```hcl
module "network" {
  source = "git::https://git.company.com/terraform-modules.git//networking/vpc?ref=v2.0.0"
}

module "database" {
  source = "git::https://git.company.com/terraform-modules.git//database/rds?ref=v1.4.1"
}
```

Rule:

- One root module per deployable unit
- Many Child Modules

---

# 25. Good Module Design

A module should have one responsibility.

Good examples:

- VPC
- RDS
- ECS
- EKS
- IAM Role
- S3 Bucket

Avoid "mega modules" that create an entire application stack.

---

# 26. Module Structure

```text
modules/
└── vpc/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── versions.tf
    ├── README.md
    └── examples/
```

Keep modules:
- Reusable
- Small
- Documented
- Versioned

Add:
- `examples/` for common usage patterns
- variable validation for safer inputs
- `sensitive = true` on outputs that should not be displayed casually

---

# 27. Variables, Locals and Outputs

## Variables

Receive input.

```hcl
variable "environment" {
  type = string
}
```

## Locals

Avoid repeated values.

```hcl
locals {
  common_tags = {
    Project = "Payments"
  }
}
```

## Outputs

Expose values.

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

Use variable validation when invalid inputs would create unsafe or expensive infrastructure.

---

# 28. Environment Strategy

Typical environments:

```text
dev
qa
stage
prod
```

Recommended:

- Separate state
- Separate backend key
- Separate pipeline
- Separate credentials or roles when environments require different access boundaries

Example backend key:

```text
payments/dev/terraform.tfstate
payments/prod/terraform.tfstate
```

---

# 29. Workspaces

Terraform workspaces allow multiple states from one configuration.

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select prod
```

Good for:
- Personal testing
- Small projects

Be careful using workspaces for shared long-lived environments such as `dev`, `stage`, and `prod`.

Large enterprises usually prefer separate directories, repositories, or explicitly separated backend keys for those environments because they are easier to understand, review, and audit.

---

# 30. Team Responsibilities

| Team | Responsibility |
|------|----------------|
| Platform | Shared modules |
| Networking | VPC, TGW, DNS |
| Security | IAM, KMS, Policies |
| Application | Uses modules |
| DevOps | CI/CD |

Application teams should consume approved modules instead of creating infrastructure from scratch.

---

# 31. Enterprise Workflow

```text
Developer
   |
Git Commit
   |
Pull Request
   |
terraform fmt
   |
terraform validate
   |
terraform plan
   |
Review
   |
Approval
   |
terraform apply
   |
Remote State Updated
```

---

# 32. Best Practices

- Organize repositories by application.
- Keep one root module per deployment.
- Reuse child modules.
- Version shared modules.
- Keep modules focused.
- Separate environments.
- Pin provider versions.
- Store secrets outside Terraform code.
- Use remote state.
- Automate with CI/CD.
- Tag all resources.
- Document module inputs and outputs.

---

# 33. Common Mistakes

- Copying Terraform between projects.
- One repository for unrelated applications.
- Giant reusable modules.
- Hardcoded values.
- No module versioning.
- Mixing networking and application resources.
- No documentation.

---

# Summary

A scalable Terraform design is based on three principles:

1. **Small reusable modules**
2. **Independent application repositories**
3. **Isolated state per environment**

Platform teams build reusable infrastructure. Application teams assemble those building blocks to deploy business applications consistently and safely.
