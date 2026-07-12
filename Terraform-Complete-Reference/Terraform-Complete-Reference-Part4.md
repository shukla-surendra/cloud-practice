# Terraform Complete Reference (Part 4)

> CI/CD, troubleshooting, enterprise workflow, interview preparation, and production best practices.

# 34. Terraform in CI/CD

A shared or production deployment should generally not rely on engineers running `terraform apply` from laptops.

Typical pipeline:

```text
Developer
    |
Git Push
    |
Pull Request
    |
terraform fmt
    |
terraform validate
    |
tflint
    |
Security Scan (Checkov/tfsec)
    |
terraform plan
    |
Review & Approval
    |
terraform apply
    |
Notification
```

## Why?

- Consistent deployments
- Audit trail
- Peer review
- Reduced production risk

Direct local applies can still be reasonable for personal sandboxes or short-lived experiments, but they are a poor default for team-owned environments.

---

# 35. Useful CI/CD Tools

| Tool | Common Usage |
|------|--------------|
| GitHub Actions | GitHub repositories |
| GitLab CI | GitLab projects |
| Jenkins | Legacy enterprises |
| AWS CodePipeline | AWS-native pipelines |
| HCP Terraform | Remote execution, state, policy, and runs |
| Atlantis | PR-based Terraform workflows |

---

# 36. Drift Detection

Drift occurs when infrastructure is changed outside Terraform.

Example:

```text
AWS Console

↓

Security Group Changed

↓

Terraform State Out of Sync
```

Detect drift by running:

```bash
terraform plan
```

Enterprise teams schedule plans daily or weekly to detect unexpected changes.

For production changes, many teams save reviewed plans and apply the exact artifact:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

---

# 37. Import Existing Resources

Terraform can manage resources created manually.

```bash
terraform import aws_s3_bucket.logs my-company-logs
```

After importing, make sure the corresponding Terraform configuration exists so future plans remain clean.

On current Terraform versions, prefer `import` blocks in configuration when possible so imports are reviewable in pull requests.

---

# 38. Useful Advanced Commands

```bash
terraform graph
terraform providers
terraform output
terraform show
terraform test
terraform state list
terraform state show
terraform state mv
terraform state rm
terraform import
```

Know these commands before working in production.

---

# 39. Troubleshooting Workflow

```text
terraform validate

↓

terraform plan

↓

Read Error Carefully

↓

Check Variables

↓

Check State

↓

Check Backend / Credentials

↓

Check Provider

↓

Fix

↓

Plan Again
```

Avoid repeatedly running `apply` until something works.

---

# 40. Enterprise Deployment Flow

```text
Platform Team
    |
Publishes Module v2.0
    |
Application Team Updates Version
    |
Pull Request
    |
Pipeline Executes Plan
    |
Approval
    |
Apply
```

Application teams should consume approved module versions rather than copying infrastructure code.

---

# 41. Enterprise Checklist

Before every deployment verify:

- Terraform version pinned
- Provider versions pinned
- Backend configured
- Remote state enabled
- State locking enabled
- Variables validated
- Plan reviewed
- Saved plan used for production apply when your workflow supports it
- Module versions fixed
- Secrets stored securely
- Resources tagged

---

# 42. Production Best Practices

## Repository

- One repository per application or platform component.
- Keep repositories focused.

## State

- One state per application and environment.
- Never share one state across unrelated systems.

## Modules

- Keep modules small.
- One responsibility per module.
- Version every shared module.

## Security

- Never commit secrets.
- Use IAM roles.
- Encrypt remote state.
- Enable state bucket versioning.
- Mark sensitive variables and outputs appropriately.

## CI/CD

- No manual production apply.
- Require approvals.
- Store plan artifacts.
- Keep logs.

---

# 43. Common Interview Questions

### Why does Terraform need a state file?

To map configuration to real infrastructure and calculate changes.

### Why use remote state?

To enable collaboration, versioning, and locking.

### Why use modules?

To reuse infrastructure and standardize deployments.

### What is the difference between `plan` and `apply`?

`plan` previews changes; `apply` executes them.

### Why avoid one huge state?

It increases deployment risk, slows plans, and prevents independent team deployments.

---

# 44. Quick Revision Sheet

Daily Commands

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
```

State Commands

```bash
terraform state list
terraform state show
terraform state mv
terraform state rm
terraform import
```

Project Structure

```text
project/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── versions.tf
└── modules/
```

Deployment Order

```text
Write
↓

Init
↓

Validate
↓

Plan
↓

Review
↓

Apply
```

---

# 45. Final Enterprise Mental Model

Think of Terraform like this:

```text
Application Repository
        |
Root Module
        |
Reusable Modules
        |
Terraform Plan
        |
Remote State
        |
Cloud Provider
        |
AWS Infrastructure
```

The application repository describes **what** should exist.

Reusable modules define **how** common infrastructure is built.

Terraform state remembers **what already exists**.

The provider communicates with the cloud APIs.

---

# Final Summary

To become productive with Terraform, remember these principles:

1. Infrastructure is defined in code.
2. Keep projects small and focused.
3. Use reusable modules.
4. Store state remotely with locking.
5. Review every plan before apply.
6. Automate deployments through CI/CD.
7. Version modules and providers.
8. Never edit state manually.
9. Separate applications, environments, and ownership boundaries.
10. Treat Terraform code like application code: review, test, document, and version it.

Following these practices will help you work effectively on both small projects and large enterprise platforms.
