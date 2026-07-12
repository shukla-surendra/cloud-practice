# Terraform Complete Reference (Part 6)

> Terraform internals, AWS communication model, Terraform vs CDK vs CloudFormation, best practices, use cases, case studies, and interview questions.

---

# 46. How Terraform Works Internally

Terraform is not just a template runner.

Internally it performs a sequence of steps:

1. Read the root module and child modules
2. Parse variables, locals, data sources, resources, and outputs
3. Resolve provider requirements and versions
4. Download or reuse provider plugins during `terraform init`
5. Read the current state from the backend
6. Build a dependency graph
7. Ask providers to refresh or read real infrastructure
8. Compare desired configuration with current state and provider responses
9. Produce an execution plan
10. Execute the graph in dependency order
11. Update state after successful operations

Think of Terraform as:

```text
HCL Configuration
      |
Terraform Core
      |
Provider Plugin (AWS)
      |
AWS APIs
      |
Real AWS Resources
```

---

# 47. How Terraform Communicates with AWS

Terraform Core does not call AWS APIs directly.

It communicates with the AWS provider plugin.

The AWS provider plugin is responsible for:

- Authenticating to AWS
- Calling AWS APIs
- Translating Terraform resource blocks into AWS API operations
- Reading current resource attributes from AWS
- Returning results back to Terraform Core

Practical flow:

```text
resource "aws_s3_bucket" "logs"
        |
Terraform Core
        |
AWS Provider Plugin
        |
AWS SDK / AWS APIs
        |
Amazon S3
```

Example:

- You write `resource "aws_instance" "web" { ... }`
- Terraform Core decides when that resource should be created
- The AWS provider plugin calls the relevant EC2 APIs
- AWS returns identifiers and attributes
- Terraform writes those results into state

This separation is why Terraform can support many providers, not just AWS.

---

# 48. Terraform Core vs Provider Plugin

## Terraform Core responsibilities

- Read configuration
- Evaluate expressions
- Build the dependency graph
- Manage state
- Create the plan
- Decide operation ordering
- Coordinate plugins over RPC

## Provider responsibilities

- Understand a specific platform such as AWS
- Authenticate to that platform
- Map Terraform resources and data sources to API calls
- Handle create, read, update, and delete behavior

This separation is central to Terraform's design.

---

# 49. Execution Model and Dependency Graph

Terraform does not just execute files top to bottom.

It builds a resource graph from references such as:

- Resource attributes
- Module outputs
- Data source usage
- Explicit `depends_on`

Example:

```text
VPC
 |
Subnets
 |
Security Group
 |
ECS Service
```

Terraform can often create independent resources in parallel, but dependent resources are executed in order.

This is one reason Terraform scales better than naive scripting.

---

# 50. Terraform vs AWS CloudFormation vs AWS CDK

These tools overlap, but they do not work the same way.

## Terraform

- Uses HCL
- Uses provider plugins
- Works across many clouds and SaaS platforms
- Keeps its own state
- Produces a plan before apply
- Strong ecosystem for multi-cloud and shared modules

## AWS CloudFormation

- AWS-native IaC service
- Uses YAML or JSON templates
- Manages infrastructure as CloudFormation stacks
- AWS handles orchestration and rollback behavior
- Best aligned with AWS-native governance patterns

## AWS CDK

- Define infrastructure in general-purpose languages such as TypeScript, Python, Java, C#, and Go
- Uses constructs and software abstractions
- Synthesizes to CloudFormation templates
- Deploys through CloudFormation
- Best when teams want application developers to use familiar languages and abstractions

Important:

> AWS CDK is not an alternative deployment engine to CloudFormation. CDK generates CloudFormation, and CloudFormation performs the deployment.

---

# 51. Terraform vs CloudFormation

## Where Terraform is stronger

- Multi-cloud support
- Same workflow for AWS, Azure, GCP, Kubernetes, and many SaaS tools
- Rich provider ecosystem
- HCL is often easier to read than large JSON/YAML templates
- Strong module model for shared platform building
- Clear planning workflow before apply

## Where CloudFormation is stronger

- Native AWS service
- Tight integration with AWS stack lifecycle
- No separate state backend for you to manage
- Strong fit for AWS-only organizations
- Natural choice where security and platform teams prefer AWS-native controls everywhere

## CloudFormation tradeoffs

- AWS only
- Large templates can become hard to maintain
- Reuse is possible, but many teams find Terraform modules more approachable than raw template composition

---

# 52. Terraform vs CDK

## Where Terraform is stronger

- Easier standardization across many teams with declarative HCL
- Better for multi-cloud or non-AWS resources
- Less application-language complexity in infra repositories
- More predictable for infra-first teams and operations-heavy organizations

## Where CDK is stronger

- Familiar programming languages
- Loops, abstractions, composition, and reuse feel natural to developers
- Higher-level constructs can reduce boilerplate
- Good fit when app and infra are owned by the same engineering team and both live in the same language ecosystem

## CDK tradeoffs

- More software-engineering complexity in infrastructure code
- Generated CloudFormation can be harder to inspect if teams do not understand synthesis
- Still limited to AWS deployment through CloudFormation

---

# 53. Which Is Best?

There is no universal winner.

The best choice depends on the operating model.

## Choose Terraform when

- You are multi-cloud
- You manage AWS plus SaaS or platform tools
- Platform teams want a standard declarative language
- Different teams need consistent modules and workflows
- You want one IaC tool across many providers

## Choose CloudFormation when

- You are AWS-only
- You want to stay fully within AWS-native tooling
- Your team is already strong in CloudFormation stacks and change sets
- You do not want to operate a separate Terraform state strategy

## Choose CDK when

- You are AWS-only or mostly AWS
- Developers prefer writing infrastructure in code, not HCL or YAML
- You want reusable high-level constructs
- Application teams own both app and infrastructure code

## Practical recommendation

- Enterprise multi-cloud platform: Terraform
- AWS-only platform team with strict AWS-native standards: CloudFormation
- AWS product team with strong software engineering culture: CDK

If you ask "which is best for most enterprise platform teams?", Terraform is often the most flexible answer.

If you ask "which is best for AWS-only developer productivity?", CDK is often the best answer.

If you ask "which is best for AWS-native managed stack orchestration?", CloudFormation is the cleanest answer.

---

# 54. Use Cases

## Terraform use cases

- Shared networking
- Multi-account AWS foundations
- Kubernetes plus cloud resources
- Cross-cloud disaster recovery
- Managing Datadog, GitHub, Cloudflare, Snowflake, or other SaaS platforms
- Standardized reusable infrastructure modules

## CloudFormation use cases

- AWS-only infrastructure managed as stacks
- Organizations standardized on AWS-native controls
- Teams using AWS service integrations that assume CloudFormation stacks

## CDK use cases

- Application teams creating AWS resources alongside app code
- Teams building reusable internal AWS constructs
- Developer-centric teams comfortable with TypeScript, Python, Java, C#, or Go

---

# 55. Small Case Studies

## Case Study 1: Startup on AWS only

Situation:

- Small team
- Single cloud
- Mostly developers
- Fast feature delivery matters more than shared platform standards

Good fit:

- AWS CDK

Why:

- Developers can stay in familiar languages
- Reusable constructs reduce boilerplate
- App and infra can live in one codebase

## Case Study 2: Mid-size company with shared platform team

Situation:

- 20+ applications
- Separate networking and security responsibilities
- Multiple environments
- Need repeatable modules and reviewable plans

Good fit:

- Terraform

Why:

- Clear module ownership
- Strong root-module and state isolation model
- Easier shared standards across many teams

## Case Study 3: Enterprise AWS-only regulated environment

Situation:

- Strict governance
- AWS only
- Change controls and standardized stack operations
- Heavy central platform oversight

Good fit:

- CloudFormation or Terraform, depending on team culture

Why:

- CloudFormation fits AWS-native operating models
- Terraform can still be better if many teams need a stronger planning workflow and reusable module ecosystem

## Case Study 4: Multi-cloud platform

Situation:

- AWS plus Azure or GCP
- Shared policy and naming standards
- Common workflow required across providers

Good fit:

- Terraform

Why:

- One tool, one language, many providers
- Consistent state and module patterns
- Lower tooling fragmentation

---

# 56. Best Practices for Tool Selection

- Optimize for operating model, not hype.
- Pick one primary IaC standard per platform area when possible.
- Avoid mixing Terraform, CDK, and CloudFormation for the same workload without a clear reason.
- Prefer Terraform when non-AWS systems matter.
- Prefer CDK when developer ergonomics in AWS matter more than cross-platform consistency.
- Prefer CloudFormation when AWS-native stack control is the main requirement.
- Standardize code review, tests, and deployment pipelines regardless of tool.

---

# 57. Terraform Best Practices for AWS

- Use remote state with locking.
- Isolate state by deployable unit and environment.
- Pin Terraform and provider versions.
- Use reusable modules for shared patterns.
- Keep root modules small and focused.
- Save reviewed plans for production changes.
- Validate variables.
- Mark sensitive variables and outputs.
- Prefer IAM roles over long-lived access keys.
- Tag everything consistently.
- Avoid ad hoc `terraform state` surgery.
- Prefer declarative `import` blocks for imports when possible.
- Run `terraform fmt`, `terraform validate`, and `terraform test` in CI/CD.

---

# 58. Interview Questions

## Conceptual

1. What is the difference between Terraform Core and a provider plugin?
2. Why does Terraform need state?
3. How does Terraform know the order in which resources should be created?
4. What is the difference between a root module and a child module?
5. Why should unrelated systems not share one state file?

## AWS-specific

1. How does Terraform communicate with AWS?
2. What is the role of the AWS provider?
3. Why is remote state important for team workflows on AWS?
4. Why is `use_lockfile = true` important in the S3 backend?
5. When would you choose CloudFormation or CDK instead of Terraform?

## Comparison

1. How is CDK different from CloudFormation?
2. How is Terraform different from CloudFormation?
3. Why might Terraform be preferred in a multi-cloud organization?
4. Why might CDK be preferred by application teams?
5. Is CDK a replacement for CloudFormation?

## Scenario-based

1. A company has 80 AWS accounts and wants common networking standards. Which tool and structure would you recommend?
2. A startup is AWS-only and all engineers write TypeScript. Would you choose Terraform or CDK, and why?
3. A bank wants strict AWS-native governance and stack-based operations. Would CloudFormation be a better fit?
4. A team already uses Terraform for AWS, Datadog, and GitHub. Should they introduce CDK for one app, or stay consistent?

---

# 59. Strong Interview Answers

## How does Terraform communicate with AWS?

Terraform Core communicates with the AWS provider plugin over RPC. The AWS provider authenticates with AWS, calls the AWS APIs, reads the results, and returns them to Terraform Core, which then updates state and executes the dependency graph.

## How is CDK different from CloudFormation?

CDK is a software framework for defining AWS infrastructure in general-purpose languages. It synthesizes CloudFormation templates, and CloudFormation performs the actual deployment.

## Which is better: CDK or Terraform?

Neither is universally better. Terraform is usually stronger for multi-cloud standardization and platform-wide consistency. CDK is usually stronger for AWS-only developer-centric teams that want infrastructure in application languages.

## Why is Terraform popular in enterprises?

Because it separates reusable modules from deployable root modules, supports many providers, has a strong planning workflow, and scales well across multiple teams and platforms.

---

# 60. Final Guidance

If you are learning for jobs or interviews:

- Understand Terraform internals at a high level
- Be able to explain provider plugins and state clearly
- Know that CDK deploys through CloudFormation
- Do not answer "which is best" with a blanket statement
- Tie your recommendation to team structure, cloud scope, and governance model

If you are building a real platform:

- Standardize on the fewest IaC tools possible
- Favor clarity over clever abstractions
- Design around ownership, state isolation, and repeatable delivery
