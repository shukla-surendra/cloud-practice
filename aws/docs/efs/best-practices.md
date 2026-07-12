# EFS — Best practices, cost, monitoring & production

> Part of the AWS Mastery track. See [PROGRESS.md](../../../PROGRESS.md).
> **Prereq:** all prior EFS docs.

Spec sections 8, 9, 10, 11.

---

## 1. Best-practice checklist
- **Encrypt at rest** (KMS) at creation; **mount with `-o tls`** (efs-utils) for in-transit.
- **Elastic throughput + General Purpose** performance mode as the default.
- **Mount target per AZ**, SG allowing **2049 only from client SGs**.
- **Access Points** for per-app/per-container isolation (enforced uid + root dir); **IAM auth + filesystem policy** to scope who mounts.
- **Lifecycle Management** to tier cold files to IA/Archive.
- Use **`amazon-efs-utils`** (handles TLS/IAM/Access Points, DNS, retries) rather than raw `nfs4`.
- **One Zone** for non-critical/dev data to cut cost; **Standard** (multi-AZ) for anything needing AZ resilience.
- Drive **parallelism** for throughput; don't expect EBS-like single-op latency.
- IaC the filesystem, mount targets, SGs, access points.

## 2. Anti-patterns
| Anti-pattern | Why it hurts | Fix |
|---|---|---|
| EFS for a transactional DB | Per-op latency kills it | EBS io2 / Block Express |
| Millions of tiny files, serial access | Metadata latency compounds | Parallelize; reconsider S3 for objects |
| Open 2049 SG | Anyone in VPC mounts your data | SG from client SG only |
| Plain `nfs4`, no TLS | Cleartext data on wire | efs-utils `-o tls` |
| Bursting on a tiny filesystem | Low baseline → "slow" | Elastic throughput |
| Everyone mounts root | No tenant isolation | Access Points |
| Never tiering | Pay hot price for cold data | Lifecycle Management |
| Single mount target | Cross-AZ latency + AZ risk | One per AZ |

## 3. Cost model [Documented]
EFS bills for **what you use**, not provisioned:
1. **Storage per GB-month by class** — Standard (highest), Standard-IA (much cheaper storage **+ per-GB retrieval fee**), Archive (cheapest + higher retrieval), and **One Zone** variants (~½ of multi-AZ). Lifecycle Management moving cold files to IA/Archive is the biggest lever.
2. **Throughput** — **Elastic**: pay **per GB read/written**; **Provisioned**: pay for the MB/s you reserve; **Bursting**: included with storage (no extra, but size-limited).
3. **Cross-AZ data**: keep clients mounting the **AZ-local** mount target to avoid cross-AZ transfer.
4. **Watch:** IA **retrieval** charges if a job suddenly scans cold data (tiering can backfire on scan-heavy access); and Provisioned throughput you no longer need.

**Cost levers, ranked:** Lifecycle→IA/Archive · One Zone for non-critical · Elastic (pay-per-use) vs over-provisioned · AZ-local mounts.

## 4. Monitoring (CloudWatch) [Documented]
- `PercentIOLimit` (General Purpose) → near 100% = ops/sec ceiling.
- `BurstCreditBalance` (Bursting) → **alarm near 0** (about to throttle to baseline).
- `MeteredIOBytes` / `TotalIOBytes`, `ClientConnections`, `PercentOfProvisioned...` (Provisioned).
- `StorageBytes` (by class — track IA vs Standard split).
- Client-side: NFS latency, `nfsiostat`, mount errors in `dmesg`.

## 5. Production patterns
- **Containers (EKS/ECS):** EFS CSI driver + **Access Points** per app → shared, persistent, multi-AZ volumes with per-app isolation. The go-to for stateful pods needing shared/RWX storage.
- **Web/CMS fleet:** shared document root/media across an autoscaling group in multiple AZs.
- **ML/analytics:** shared training data/checkpoints read by many workers (though **FSx for Lustre** wins for extreme HPC throughput).
- **Lift-and-shift NAS:** replace an on-prem NFS filer with minimal app change.
- **Home directories / dev shares:** POSIX permissions, per-user Access Points.

## 6. Choosing EFS vs EBS vs FSx vs S3
- Shared **POSIX Linux** files, multi-AZ, elastic → **EFS**.
- One instance's disk / DB → **EBS**.
- Windows/SMB, HPC Lustre, ONTAP/OpenZFS features → **FSx**.
- Objects / data lake / backups → **S3**.

---

## Sources
- AWS docs: *EFS pricing*, *Lifecycle management*, *EFS CloudWatch metrics*, *EFS CSI driver*, *Choosing storage*.
- Well-Architected: *Cost* & *Performance* pillars; *Storage services comparison*.

---

## Self-check
1. Rank the top three cost levers for a mostly-cold EFS dataset, and name the gotcha of the biggest one.
2. Which throughput + performance mode combo is the modern default, and why?
3. For EKS pods needing shared RWX storage with per-app isolation, what's the pattern (two features)?
4. Which two metrics do you alarm on, and what does each protect against?
5. Give a one-line rule for EFS vs EBS vs S3 vs FSx.
