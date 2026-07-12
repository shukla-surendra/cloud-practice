# AWS Mastery — Master Progress & Resume Tracker

> **This file is the single source of truth for the whole journey.**
> To resume in any new session, read this file first. It says exactly where we are,
> what's next, and how the learning contract works. Everything else is detail.

**Owner:** Surendra Shukla · **Started:** 2026-07-12 · **Mode:** one service at a time, architecture/internals depth (NOT cert-oriented).

---

**Repo layout:** two cloud tracks — `aws/` (active) and `gcp/` (planned). Each holds `docs/`, `quizzes/`, `terraform/`, `labs/`, etc. Shared docs tooling lives in `scripts/` + `Makefile` at the root (`make docs` renders all Markdown to a themed HTML site; `make check` validates links).

## 0. How to resume (read this first every session)

1. Open this file → find **Current Position** (§3).
2. Open the current service's docs under `<cloud>/docs/<service>/` and the open gate under `<cloud>/quizzes/<service>/`.
3. If a gate is **OPEN**, the learner answers the gate questions; the mentor grades, patches gaps, then advances.
4. Mentor updates §3 (Current Position) and §4 (Changelog) after every module/gate.

**Learning contract (do not violate):**
- One service at a time. **Do NOT advance to the next module until the current gate is cleared.**
- Each service is delivered as ~4–6 **progressive gated modules** (the 21-section spec below, chunked for retention — never a single 20k-word dump).
- Tag every internal claim **[Documented]** (docs / re:Invent / patents / whitepapers) vs **[Inferred]** (reconstruction from behavior). The learner must never mistake reconstruction for fact.
- Relate every AWS internal back to **Linux, Kubernetes, networking, and distributed-systems** primitives the learner already knows.
- **Why before How**, always. Never oversimplify, never skip internals.
- Scaffold the repo **incrementally** — create folders/files as each service/module is covered, not empty placeholders upfront.
- After each module: stop, quiz (conceptual + scenario + predict-behavior), wait for answers, grade, then continue.

---

## 1. The 21-section teaching spec (applied to every service)

Every service is taught to this depth, distributed across its modules:

1. Why the service exists (problem, history, prior solutions, why insufficient, why AWS built it, what if it didn't exist)
2. Internal architecture (request flow, networking, storage, metadata, control/data plane, replication, scaling, availability, consistency, failure recovery, multi-region, HA, fault tolerance)
3. How AWS built it (internal services, distributed-systems concepts, databases, storage engines, consensus, queues, networking, scheduling, caching, load balancing, security)
4. Deep networking (packet flow, DNS, routing, TCP, TLS, NAT, IGW, private networking, ENI, SGs, NACLs)
5. Storage architecture (physical/logical, placement, replication, partitioning, sharding, compression, encryption, perf)
6. Security (IAM, authN, authZ, encryption, KMS, secrets, certs, least privilege, cross-account, threat models)
7. Performance (throughput, latency, limits, bottlenecks, scaling, benchmarking, cost vs perf)
8. Real production architecture (Netflix/Uber/Airbnb/Amazon/Spotify/Databricks/banks/healthcare/e-commerce)
9. Best practices (prod configs, mistakes, anti-patterns, design/security recs, operational excellence)
10. Cost optimization (billing, pricing, hidden costs, monitoring, savings, reserved, spot, lifecycle)
11. Monitoring (CloudWatch, logs, metrics, tracing, dashboards, alarms, EventBridge, Config, CloudTrail)
12. Hands-on labs (beginner→expert + architecture/production/failure/perf/cost/security; each: objectives, architecture, implementation, validation, cleanup)
13. Coding (Python/Boto3 preferred, Terraform, AWS CLI, CloudFormation, CDK-Python, Shell)
14. Debugging (common failures, diagnosis, CloudWatch/logs/metrics, CLI, networking, IAM, perf)
15. Interview prep (junior/senior/principal, architecture, scenario, incident)
16. Comparison (Azure, GCP, Kubernetes, traditional infra, open-source alternatives)
17. Internal implementation (CAP, Paxos/Raft, leader election, consistent hashing, quorum, caching, distributed locking, eventual consistency, vector clocks, LSM/B-trees, bloom filters, WAL, object storage, network/virtualization, hypervisors, Firecracker, Nitro)
18. Sources (AWS docs, GitHub, blogs, whitepapers, research papers, eng blogs, OSS, RFCs, books)
19. Visual learning (architecture/flow/sequence/network/storage diagrams, comparison tables, mind maps)
20. Knowledge check (conceptual + scenario + debugging; gate — don't continue until correct)
21. Repo structure (docs / diagrams / terraform / cloudformation / cdk / boto3 / python / labs / quizzes / cheatsheets / notes)

---

## 2. Curriculum roadmap (service order)

Order is chosen for dependency + distributed-systems richness. Adjustable anytime.
**AWS is the primary track; the GCP track (in `gcp/`) starts later and is taught largely by contrast with AWS.**

### AWS track (`aws/`)

| # | Service | Status | Notes |
|---|---------|--------|-------|
| 1 | **VPC / Networking** | 🟡 IN PROGRESS | Foundation for everything. See per-module plan below. |
| 2 | IAM | ⬜ Planned | Security substrate under every service. |
| 3 | S3 | ⬜ Planned | Object storage, durability math, erasure coding, consistency. |
| 4 | EC2 / Nitro | ⬜ Planned | Compute, Nitro cards, Firecracker, virtualization. |
| 5 | EBS | ⬜ Planned | Block storage, replication. |
| 6 | Route 53 | ⬜ Planned | DNS internals, health checks, routing policies. |
| 7 | ELB (ALB/NLB) | ⬜ Planned | Load balancing internals, Hyperplane. |
| 8 | RDS / Aurora | ⬜ Planned | Aurora storage-compute separation. |
| 9 | DynamoDB | ⬜ Planned | Consistent hashing, quorum, streams. |
| 10 | Lambda | ⬜ Planned | Firecracker microVMs, cold starts. |
| … | (KMS, SQS/SNS, CloudFront, ECS/EKS, Kinesis, CloudWatch, Step Functions, …) | ⬜ Backlog | Sequenced later. |

### GCP track (`gcp/`)

| # | Service | Status | Notes |
|---|---------|--------|-------|
| — | Starts after AWS core (networking/IAM/storage/compute) | ⬜ Planned | Taught by contrast: GCP VPC (global) vs AWS VPC, GCP IAM (resource hierarchy) vs AWS IAM, GCS vs S3, GCE vs EC2. See `gcp/README.md`. |

Legend: ⬜ Planned · 🟡 In progress · ✅ Complete

---

## 3. CURRENT POSITION  ← resume here

- **Cloud / Service:** AWS · VPC / Networking (service #1)
- **Content written so far:** M1 `aws/docs/vpc/architecture.md` · M2 `aws/docs/vpc/networking.md` + runnable Terraform `aws/terraform/vpc/` (3-tier multi-AZ VPC).
- **Open gates (awaiting learner answers):** `aws/quizzes/vpc/module-1-gate.md` (4 Q) · `aws/quizzes/vpc/module-2-gate.md` (6 Q, incl. Terraform).
- **Note:** learner asked to expand docs + Terraform ahead of the gated walkthrough — M1/M2 content is on disk as reference; gates still open for the interactive check.
- **Next:** **M3 — Security Groups vs NACLs internals · ENI deep-dive · Nitro enforcement · VPC Flow Logs** → `aws/docs/vpc/security.md`, `aws/docs/vpc/internals.md`. Also pending: user may want to answer gates and/or `terraform validate` the example.

### VPC per-module plan

| Module | Topic | Target file(s) | Status |
|--------|-------|----------------|--------|
| M1 | Why VPC exists · two-networks mental model · internal architecture (Mapping Service, Nitro data plane, Blackfoot edge, distributed stateful SGs) | `aws/docs/vpc/architecture.md` | ✅ Delivered · gate OPEN |
| M2 | Deep packet flow · DNS · routing · IGW/NAT · peering/TGW/endpoints/PrivateLink · **Terraform 3-tier VPC** | `aws/docs/vpc/networking.md`, `aws/terraform/vpc/` | ✅ Written · gate OPEN (`module-2-gate.md`) |
| M3 | Security Groups vs NACLs internals · ENI deep-dive · Nitro enforcement · VPC Flow Logs | `aws/docs/vpc/security.md`, `aws/docs/vpc/internals.md` | ⬜ |
| M4 | Advanced connectivity · multi-region/HA · real production architectures · cost | `aws/docs/vpc/best-practices.md` | ⬜ |
| M5 | Labs · Terraform/CDK/boto3/CLI · debugging (Reachability Analyzer, Flow Logs) | `aws/labs/vpc/`, `aws/terraform/vpc/`, `aws/boto3/vpc/`, `aws/docs/vpc/troubleshooting.md` | ⬜ |
| M6 | Interview drills · Azure VNet / GCP VPC / K8s CNI comparison · consolidation | `aws/docs/vpc/interview.md`, `aws/cheatsheets/vpc.md` | ⬜ |

---

## 4. Changelog

- **2026-07-12** — Project kicked off. Chose VPC as service #1, progressive gated modules, scaffold-as-we-go. Delivered VPC M1 (`aws/docs/vpc/architecture.md`); opened M1 gate.
- **2026-07-12** — Reorganized into `aws/` + `gcp/` tracks (moved docs under `aws/`). Copied docs tooling from the `python-debugging` repo — `scripts/build_docs.py`, `scripts/check_links.py`, `Makefile`, `.gitignore` — and rebranded the generated site to "AWS & GCP Mastery". Added `requirements.txt`.
- **2026-07-12** — Expanded AWS docs: wrote VPC **M2** (`networking.md` — building blocks, routing, IGW/NAT/endpoints/peering/TGW/DNS packet flows) + a runnable **Terraform 3-tier VPC** (`aws/terraform/vpc/`, 8 files). Added M2 gate (6 Q). Added Terraform ignores to `.gitignore`. Links validated (8 files), site builds. Terraform not installed locally → not `validate`-d yet.

---

## 5. Repo structure (filled incrementally)

```
cloud-practice/
├── PROGRESS.md                 # THIS FILE — master tracker / resume anchor
├── README.md                   # overview + how to use
├── requirements.txt            # docs tooling deps (markdown-it-py)
├── Makefile                    # `make docs` (render+serve), `make check` (link-check+build)
├── scripts/
│   ├── build_docs.py           # renders every *.md in repo → themed HTML in docs_html/
│   └── check_links.py          # validates relative Markdown links (CI-friendly)
├── aws/                        # ── AWS track (active) ──
│   ├── docs/<service>/
│   │   ├── architecture.md     # why + mental model + internal architecture
│   │   ├── internals.md        # distributed-systems internals / algorithms
│   │   ├── networking.md       # deep packet flow, DNS, routing
│   │   ├── security.md         # IAM, SG/NACL, encryption, threat models
│   │   ├── best-practices.md   # prod configs, anti-patterns, cost, prod architectures
│   │   ├── troubleshooting.md  # debugging, common failures, diagnosis
│   │   └── interview.md        # junior→principal Q&A, scenarios
│   ├── diagrams/<service>/     terraform/<service>/   cloudformation/<service>/
│   ├── cdk/<service>/          boto3/<service>/        python/<service>/
│   ├── labs/<service>/         quizzes/<service>/      cheatsheets/   notes/
└── gcp/                        # ── GCP track (planned) — mirrors aws/ ──
    └── README.md
```
Folders are created only when a service/module needs them (scaffold-as-we-go).
`docs_html/`, `.venv/`, and `__pycache__/` are git-ignored (generated / local).
