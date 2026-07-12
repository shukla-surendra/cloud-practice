# Terraform Complete Reference (Part 5)

> Goal: organize Terraform so it scales from a single engineer to many teams without turning into a monolith.

---

# Why Project Organization Matters

Writing Terraform is straightforward.

Operating Terraform across many applications, environments, and teams is where most failures happen.

A good structure should:

- Be easy to understand
- Minimize merge conflicts
- Support multiple environments
- Keep state isolated
- Match team ownership
- Encourage reuse
- Reduce deployment blast radius

---

# Start Simple

A small project usually starts like this:

```text
terraform-project/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── terraform.tfvars
└── README.md
```

Typical file responsibilities:

| File | Purpose |
|------|---------|
| `main.tf` | Primary resources |
| `provider.tf` | Provider configuration |
| `variables.tf` | Input variables |
| `outputs.tf` | Exported values |
| `versions.tf` | Terraform and provider version constraints |
| `terraform.tfvars` | Variable values for a specific use case |

Terraform loads all `.tf` files in the directory as a single root module, so file splits are mainly for readability and ownership, not execution order.

Example:

```hcl
provider "aws" {
  region = "ap-south-1"
}

variable "instance_type" {
  type = string

  validation {
    condition     = contains(["t3.micro", "t3.small"], var.instance_type)
    error_message = "Use an approved instance type."
  }
}

resource "aws_instance" "web" {
  ami           = "ami-xxxxxxxx"
  instance_type = var.instance_type
}

output "instance_id" {
  value = aws_instance.web.id
}
```

---

# As the Project Grows

Once `main.tf` becomes crowded, split files by concern:

```text
terraform-project/
├── provider.tf
├── versions.tf
├── variables.tf
├── outputs.tf
├── network.tf
├── compute.tf
├── database.tf
├── security.tf
└── terraform.tfvars
```

This improves readability, but it does not change the fact that the directory is still one root module and one deployment unit.

---

# Terraform Building Blocks

A Terraform project is usually composed of:

```text
Root Module
├── Provider Configuration
├── Resources
├── Variables
├── Outputs
├── Child Modules
└── State
```

Key concepts:

- A root module is the directory where Terraform commands run.
- Child modules are reusable building blocks called from the root module.
- State tracks the resources Terraform manages.

---

# The Biggest Structural Mistake

Many teams begin with an environment-first layout:

```text
terraform/
├── dev/
├── qa/
├── prod/
├── main.tf
└── variables.tf
```

This can work for a demo.

It usually breaks down in larger organizations because it often leads to:

- Oversized state files
- Poor ownership boundaries
- Slower plans
- Larger blast radius
- More merge conflicts

If one team changes a shared state for a security group, another team can be blocked even if they are deploying a different application.

The real lesson is: scope Terraform state to logically related, independently deployable infrastructure.

---

# Think in Deployable Units

A better mental model is to organize around projects, workloads, or platform domains.

```text
terraform/
├── customer-portal/
├── payments/
├── analytics/
├── networking/
├── security/
└── shared-services/
```

Each directory should represent one deployable root module or a clearly grouped set of root modules.

Environment separation still matters, but it usually lives inside each workload through separate root directories, backend keys, pipelines, or repositories.

---

# Root Modules and Child Modules

Every deployable unit has one root module.

```text
payments/
├── main.tf
├── variables.tf
└── outputs.tf
```

That root module should assemble reusable modules instead of duplicating infrastructure everywhere:

```hcl
module "vpc" {
  source = "git::https://git.company.com/terraform-modules.git//networking/vpc?ref=v2.0.0"
}

module "database" {
  source = "git::https://git.company.com/terraform-modules.git//database/rds?ref=v1.4.1"
}
```

Practical rule:

- One root module per deployable unit
- Many child modules when reuse makes sense

---

# Why Separate Reusable Modules

If five applications all need similar networking, IAM, or database patterns, copy-pasting those resources into every root module creates drift and review fatigue.

Better:

```text
terraform-modules/
├── vpc/
├── iam-role/
├── rds/
└── s3-bucket/
```

Benefits:

- Less duplication
- Easier maintenance
- More consistent security controls
- Cleaner reviews
- Safer upgrades through module versioning

Avoid giant "do everything" modules. A module should usually have one clear responsibility.

---

# State Strategy

One of the most important rules is:

> One independently deployable root module and environment should have one state file.

Example:

```text
payments/dev/terraform.tfstate
payments/prod/terraform.tfstate
analytics/prod/terraform.tfstate
networking/prod/terraform.tfstate
```

Do not put the whole company into one giant state.

Smaller states:

- Reduce blast radius
- Improve concurrency
- Make plans faster
- Simplify approvals and ownership

A repository can still contain multiple root modules as long as those deployment boundaries remain clear.

---

# Environment Organization

Most real systems need multiple environments:

```text
dev
qa
uat
stage
prod
dr
```

Common ways to model them:

## Option 1: Separate directories

```text
payments/
├── dev/
├── qa/
├── stage/
└── prod/
```

## Option 2: Separate repositories

```text
payments-dev-infra/
payments-prod-infra/
```

## Option 3: Same repository, separate backend keys and pipelines

```text
payments/
└── live/
    ├── dev/
    └── prod/
```

Typical environment differences:

- Variable values
- Backend key or state location
- Pipeline approvals
- Resource sizing
- Integrations and access controls

Terraform workspaces can be useful for personal testing or small setups, but most larger teams prefer explicit directories, repositories, or backend separation for long-lived shared environments.

This matches the Terraform CLI guidance that workspaces are not a substitute for separate credentials and access boundaries.

---

# Repository Layout Options

There is no single correct repository structure. Pick one that matches team size and ownership.

## Option 1: Monorepo

```text
company-infra/
├── networking/
├── security/
├── shared-services/
├── payments/
├── hr/
└── analytics/
```

Pros:

- Easy to discover everything
- Easier large-scale refactoring

Cons:

- More merge conflicts
- Broader CI/CD scope
- Harder ownership boundaries

Often best for smaller platform teams.

## Option 2: Repository per project or platform domain

```text
terraform-modules/
networking-infra/
security-infra/
payments-infra/
customer-portal-infra/
analytics-infra/
```

Pros:

- Independent deployments
- Clear ownership
- Smaller pipelines
- Easier access control

Cons:

- More repositories to manage

This is a common enterprise model because it maps well to team ownership and release cycles.

---

# Enterprise Case Study

Imagine an organization with:

- 100+ applications
- Multiple business units
- Shared networking and security teams
- Platform engineers maintaining reusable infrastructure
- Development, QA, UAT, production, and disaster recovery environments

The company wants infrastructure to be:

- Version controlled
- Repeatable
- Auditable
- Secure
- Automated

A practical enterprise layout often looks like this:

```text
company/
├── terraform-modules/
│   ├── networking/
│   ├── compute/
│   ├── storage/
│   ├── database/
│   └── security/
├── platform-live/
│   ├── networking/
│   ├── security/
│   ├── identity/
│   └── shared-services/
├── applications/
│   ├── hr/
│   ├── finance/
│   ├── payments/
│   ├── analytics/
│   └── customer-portal/
└── docs/
```

Typical ownership:

| Repository or Area | Owner |
|--------------------|-------|
| `terraform-modules` | Platform Team |
| `networking` | Network Team |
| `security` | Security Team |
| `shared-services` | Platform Team |
| `payments` | Payments Team |
| `customer-portal` | Customer Portal Team |

Terraform structure should reflect organizational boundaries, approval paths, and deployment ownership.

---

# Recommended Enterprise Flow

```text
Developer
  |
Pull Request
  |
terraform fmt
  |
terraform validate
  |
terraform plan
  |
Review and Approval
  |
terraform apply
  |
Remote State Updated
  |
Cloud Resources Updated
```

For shared or production environments, local `terraform apply` from laptops should not be the default operating model.

Personal sandboxes are different. Shared environments need consistent pipelines, logs, approvals, and repeatable execution.

When approvals matter, save the reviewed plan artifact and apply that exact plan rather than rerunning an unsaved plan during deployment.

---

# Design Principles

- Organize by ownership and deployability first.
- Keep environments explicit.
- Keep state small and isolated.
- Separate reusable modules from live infrastructure.
- Version shared modules.
- Reuse patterns instead of copying code.
- Keep repository boundaries clear.
- Standardize layouts across teams when possible.
- Treat Terraform like production code with review and CI/CD.
- Prefer declarative imports and reviewed migrations over ad hoc state surgery.

---

# Common Mistakes

- One state file for unrelated systems
- One huge repository with unclear ownership
- Copy-pasting Terraform between applications
- Giant modules that build entire platforms
- Hardcoded values and credentials
- Mixing platform, networking, and application concerns without boundaries
- Using workspaces as the primary strategy for all shared environments
- Running production changes without review

---

# Summary

Terraform scales well when:

- Root modules match deployable units
- States are isolated per root module and environment
- Shared infrastructure is versioned as reusable modules
- Repository structure matches team ownership
- Delivery happens through repeatable pipelines

The goal is not just tidy folders. The goal is a structure that lets many engineers change infrastructure safely at the same time.
