# EFS Cheatsheet

One-page recall. Full detail in [`../docs/efs/`](../docs/efs/README.md).

## Mental model
**Managed, elastic, multi-AZ NFS (POSIX) filesystem.** Mount via a **per-AZ mount target** (an ENI/IP on TCP 2049). Every op = network round trip → higher per-op latency than EBS. Pay for **used** GB (opposite of EBS).

## EBS vs EFS vs S3
| | EBS | **EFS** | S3 |
|---|---|---|---|
| Interface | block | **file (NFS/POSIX)** | object |
| Sharing | 1 instance | **many, concurrent** | any (API) |
| AZ | single | **multi-AZ** (Standard) | regional |
| Capacity | provisioned | **elastic** | elastic |
| Latency | low | ms/op | highest |

## Storage classes
Standard (multi-AZ) · Standard-IA (cheap storage + retrieval fee) · Archive (cheapest, slow) · **One Zone** variants (~½ cost, 1 AZ). **Lifecycle Management** tiers cold files.

## Performance
- **Performance mode:** General Purpose (default, low latency) vs Max I/O (legacy, higher latency).
- **Throughput mode:** **Elastic** (auto, pay-per-use — default) · Bursting (scales w/ size; small FS = slow; watch `BurstCreditBalance`) · Provisioned (fixed MB/s).
- Weak spot = **tiny files / metadata / serial**. Fix: parallelize, bigger I/O. Not for DBs.

## Security
- **Mount target SG: allow 2049 only from client SG** (never 0.0.0.0/0). NACL ephemerals too.
- **Encrypt at rest** (KMS, at creation). **In transit:** efs-utils `-o tls` (stunnel). Enforce via filesystem policy (`aws:SecureTransport`).
- **Access Points** = enforce POSIX uid/gid + root dir → per-app/tenant isolation (EKS/ECS pattern).
- **IAM auth:** mount `-o iam`; filesystem resource policy scopes who mounts.

## Mount (Linux)
```bash
sudo yum install -y amazon-efs-utils      # or apt
sudo mkdir /mnt/efs
sudo mount -t efs -o tls,iam fs-0abc123:/ /mnt/efs
# via Access Point:
sudo mount -t efs -o tls,accesspoint=fsap-0xyz fs-0abc123:/ /mnt/efs
# fstab: fs-0abc123:/ /mnt/efs efs _netdev,tls 0 0
```

## Cost levers (ranked)
Lifecycle→IA/Archive · One Zone for non-critical · Elastic vs over-provisioned · AZ-local mounts. Watch **IA retrieval** on scans.

## Metrics to alarm
`BurstCreditBalance`~0 (Bursting throttle) · `PercentIOLimit`~100% (GP ops ceiling) · `StorageBytes` by class · `ClientConnections`.

## Debug "mount hangs"
SG 2049 from client? → mount target in client AZ? → NACL ephemerals? → DNS (VPC dns on)? → efs-utils installed? → policy requires tls/iam? Test: `nc -vz fs-xxx.efs.region.amazonaws.com 2049`.

## When NOT EFS
DB → EBS io2 · Windows/SMB → FSx Windows · HPC → FSx Lustre · objects → S3 · single-instance disk → EBS.

## Terraform / boto3
`aws_efs_file_system` · `aws_efs_mount_target` (per subnet) · `aws_efs_access_point` · `aws_efs_file_system_policy` · SG for 2049. Examples: [`../terraform/efs/`](../terraform/efs/README.md), [`../boto3/efs/`](../boto3/efs/README.md).
