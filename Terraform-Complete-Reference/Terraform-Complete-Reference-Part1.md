# Terraform Complete Reference (Part 1)

> A concise practical reference from basic to enterprise concepts.

## 1. What is Terraform?
Terraform is an Infrastructure as Code (IaC) tool that provisions and manages infrastructure using declarative configuration.

**Benefits**
- Version controlled
- Repeatable
- Automated
- Reviewable
- Multi-cloud

---

## 2. Terraform Architecture

```text
Terraform CLI
     |
Configuration (.tf)
     |
 Provider
     |
 Cloud API (AWS/Azure/GCP)
```

Components:
- CLI
- Provider
- State
- Configuration
- Backend

---

## 3. Terraform Lifecycle

```text
Write Code
    ↓
terraform init
    ↓
terraform fmt
    ↓
terraform validate
    ↓
terraform plan
    ↓
Review
    ↓
terraform apply
    ↓
Infrastructure
```

---

## 4. Project Structure

```text
project/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── terraform.tfvars
└── modules/
```

---

## 5. Important Files

| File | Purpose |
|------|---------|
| main.tf | Resources |
| provider.tf | Cloud provider |
| variables.tf | Input variables |
| outputs.tf | Export values |
| versions.tf | Version pinning |
| terraform.tfvars | Variable values |

---

## 6. Common Commands

```bash
terraform init
terraform fmt
terraform validate
terraform test
terraform plan
terraform apply
terraform destroy
terraform output
terraform show
terraform version
terraform providers
```

### State Commands

```bash
terraform state list
terraform state show
terraform state mv
terraform state rm
terraform import
```

---

## 7. Internal Working

1. Read configuration
2. Download providers
3. Read state
4. Query cloud APIs
5. Build dependency graph
6. Calculate execution plan
7. Apply changes
8. Update state

---

## 8. Terraform State

State records Terraform's bindings to the real infrastructure it manages.

Never edit `terraform.tfstate` manually.

Enterprise best practice:
- Remote backend
- Versioning
- Encryption
- Locking

For AWS S3 backends on newer Terraform versions, prefer S3 lockfiles with `use_lockfile = true`. Older DynamoDB-based locking still exists in some estates, but it is deprecated.

---

## 9. Modules

- One root module per deployment
- Many reusable child modules
- Platform team owns shared modules

Example:

```hcl
module "vpc" {
  source = "../modules/vpc"
}
```

---

## 10. Enterprise Repository

```text
terraform-modules/
applications/
networking/
security/
shared-services/
```

---

## Best Practices

- Keep modules small
- Review every plan
- Save reviewed plans for production applies
- Pin provider versions
- Store state remotely
- One state per application/environment
- Never commit secrets
- Use meaningful names
- Reuse modules
- Separate networking from applications
- Add variable validation and mark sensitive values
- Use CI/CD

---

## Common Mistakes

- Giant state file
- One huge main.tf
- Hardcoded values
- No module versioning
- Secrets in Git
- Manual state edits
- Applying without reviewing the plan

---

## Summary

Terraform scales well when you:
- Organize by application
- Reuse modules
- Keep state isolated
- Automate deployments
- Follow consistent project structure
