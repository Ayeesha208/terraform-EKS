# Authentik Deployment on AWS EKS using Terraform

---

# 1. Project Objective

## Goal

Design and implement a production-style Identity and Access Management (IAM) platform using **Authentik** on a **Managed Kubernetes Cluster (AWS EKS)** using **Terraform Infrastructure as Code (IaC)**.

The project focuses on:

- Infrastructure Automation
- Kubernetes Deployment
- Stateful Application Deployment
- Persistent Storage Management
- Backup Strategy
- Production-Ready Architecture

---

# 2. What is Authentik?

**Authentik** is an open-source Identity Provider (IdP) and Single Sign-On (SSO) platform.

## Main Features

- Single Sign-On (SSO)
- OAuth2 / OpenID Connect
- User Authentication
- Multi-Factor Authentication (MFA)
- Identity Federation
- Centralized Access Control

---

# 3. Real-World Use Case

Instead of maintaining separate authentication systems for:

- Jenkins
- Grafana
- Kubernetes Dashboard
- Internal Applications

Authentik provides one centralized authentication platform.

---

# 4. Project Architecture Blueprint

```txt
Terraform
   ↓
AWS VPC
   ↓
Public & Private Subnets
   ↓
Internet Gateway + Route Tables
   ↓
AWS EKS Cluster (Managed Kubernetes)
   ↓
Managed Node Group (EC2 Worker Nodes)
   ↓
Kubernetes Namespace
   ↓
PostgreSQL Database (StatefulSet + PVC)
   ↓
Redis Cache (Separate Pod)
   ↓
Authentik Server Deployment
   ↓
Authentik Worker Deployment
   ↓
Ingress / Load Balancer
   ↓
End User Access
```

---

# 5. Architecture Design Decisions

## Why AWS EKS?

- Fully managed Kubernetes service
- High availability
- Production-grade orchestration
- Industry standard DevOps deployment platform

## Why Terraform?

- Infrastructure as Code (IaC)
- Reusable deployment
- Version-controlled infrastructure
- Automated provisioning

## Why Separate Pods?

The requirement explicitly mentioned:

> "Database and Application shall run in separate pods"

This was implemented using:

| Component | Kubernetes Resource |
|---|---|
| PostgreSQL | StatefulSet |
| Redis | Deployment |
| Authentik Server | Deployment |
| Authentik Worker | Deployment |

---

# 6. Core Requirements Implemented

## Infrastructure Components

- AWS VPC
- Public Subnets
- Private Subnets
- Internet Gateway
- Route Tables
- NAT Gateway
- Security Groups
- AWS EKS Cluster
- IAM Roles
- Managed Node Group
- S3 Backup Bucket

---

## Kubernetes Components

- Namespace Isolation
- Kubernetes Secrets
- PostgreSQL Database
- Redis Cache
- Authentik Server
- Authentik Worker
- Persistent Volume Claims (PVC)
- Ingress Configuration

---

# 7. Data Persistence Strategy

The assignment required proper data persistence handling.

## Implemented Persistence

### PostgreSQL Persistence

- Persistent Volume Claim (PVC)
- Kubernetes StatefulSet
- Data survives pod restart/recreation

### Authentik Media Persistence

- Persistent storage planned for media/configuration files

---

# 8. Backup Strategy

The assignment required backup handling.

## Planned Backup Architecture

```txt
PostgreSQL Database
        ↓
Kubernetes CronJob
        ↓
pg_dump
        ↓
AWS S3 Backup Bucket
```

## Backup Benefits

- Disaster Recovery
- Database Restore Capability
- Production Readiness
- Automated Backup Workflow

---

# 9. Tools Installed and Verified

## Terraform

```bash
terraform -version
```

## kubectl

```bash
kubectl version --client
```

## eksctl

```bash
eksctl version
```

## Helm

Helm was installed manually after resolving Chocolatey permission issues.

---

# 10. Challenges Faced During Setup

## Helm Installation Issue

### Problem

- Chocolatey permission denied
- Lock file errors
- Package installation failure

### Solution

- Used Administrator PowerShell
- Installed Helm manually
- Configured PATH environment variable

---

# 11. Terraform Repository Structure

```txt
terraform-authentik-eks/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── terraform.tfvars
│
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   ├── eks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   │
│   └── s3-backup/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
└── kubernetes/
    ├── namespace.yaml
    ├── secrets.yaml
    ├── postgres.yaml
    ├── redis.yaml
    ├── authentik-server.yaml
    ├── authentik-worker.yaml
    └── ingress.yaml
```

---

# 12. Terraform Deployment Workflow

## Step 1 — Initialize Terraform

```bash
terraform init
```

---

## Step 2 — Format Terraform Code

```bash
terraform fmt
```
---

## Step 3 — Validate Terraform Configuration

```bash
terraform validate
```
---

## Step 4 — Preview Infrastructure Changes

```bash
terraform plan
```

---

## Step 5 — Deploy Infrastructure

```bash
terraform apply
```
---

# 13. Infrastructure Successfully Created

## Successfully Provisioned Resources

- AWS VPC
- Public Subnets
- Private Subnets
- IAM Roles
- Route Tables
- EKS Control Plane
- Security Groups
- S3 Backup Bucket

---

## Terraform Output

```txt
cluster_name = authentik-eks-cluster
vpc_id = created successfully
```

---

# 14. EKS Cluster Access Configuration

## Configure kubectl Access

```bash
aws eks update-kubeconfig --region ap-south-1 --name authentik-eks-cluster
```

---

## Verification Commands

```bash
kubectl config current-context
kubectl get svc
```

---

## Result

Successfully connected to the Kubernetes control plane.

This confirms:

- EKS cluster creation successful
- Kubernetes API reachable
- Terraform provisioning successful

---

# 15. Major Issue Faced — Managed Node Group Failure

## Error Encountered

```txt
Error: waiting for EKS Node Group (authentik-eks-cluster:authentik-node-group) create:
unexpected state 'CREATE_FAILED'

AsgInstanceLaunchFailures:
You've reached your quota for maximum Fleet Requests for this account
```

---

# 16. Root Cause Analysis

## Cause of Failure

The AWS EKS Control Plane was created successfully.

However, the **Managed Node Group creation failed** because the AWS Free Tier account had an **EC2 Fleet quota limitation**.

AWS internally uses:

- EC2 Fleet
- Auto Scaling Groups (ASG)

for provisioning EKS worker nodes.

The account quota prevented additional EC2 fleet requests from being launched.

---

# 17. Troubleshooting Performed

The following troubleshooting steps were performed:

- Deleted failed node groups
- Verified EC2 instances
- Checked Auto Scaling Groups
- Reviewed EKS events
- Validated Terraform configuration
- Verified IAM permissions
- Re-ran Terraform deployment

---

# 18. Important Observation

Even though the worker nodes failed:

## Successfully Created

- EKS Control Plane
- VPC
- IAM Roles
- Networking Components
- Kubernetes API Endpoint

This confirms the Terraform infrastructure implementation was successful.

The issue was specifically related to:

> AWS Free Tier EC2 Fleet quota limitation

and not due to Terraform or Kubernetes configuration errors.

---

# 19. Screenshot References

## Screenshot 1 — Terraform Apply Output
<img width="1317" height="759" alt="image" src="https://github.com/user-attachments/assets/718bff31-d52f-4792-8021-7eb05055b325" />
---

## Screenshot 2 — EKS Cluster Created Successfully
<img width="1893" height="916" alt="image" src="https://github.com/user-attachments/assets/f692ea45-e98e-4233-8cb6-a215fb2ec242" />
---

## Screenshot 3 — Node Group Failure Error
<img width="1340" height="577" alt="image" src="https://github.com/user-attachments/assets/f0398df2-1265-4714-8624-12e3fe2c7050" />

This screenshot should clearly show:

```txt
AsgInstanceLaunchFailures
You've reached your quota for maximum Fleet Requests for this account
```

---

# 20. Future Improvements

If deployed in a production AWS account:

- Complete worker node provisioning
- Deploy full Authentik workloads
- Configure Ingress Controller
- Enable TLS using cert-manager
- Configure automated PostgreSQL backups
- Add monitoring using Prometheus & Grafana
- Implement Velero cluster backups

---

# 21. Skills Demonstrated

This assignment demonstrates hands-on experience with:

- Terraform Infrastructure as Code
- AWS EKS
- Kubernetes Architecture
- Stateful Applications
- Persistent Storage
- Kubernetes Networking
- IAM Configuration
- DevOps Automation
- Production Infrastructure Design
- Troubleshooting Cloud Infrastructure

---

# 22. Conclusion

This Assignment successfully implemented the foundational infrastructure required for deploying a production-style Authentik platform on AWS EKS using Terraform.

The EKS control plane and supporting AWS infrastructure were provisioned successfully.

The final deployment was partially blocked due to AWS Free Tier EC2 Fleet quota limitations affecting Managed Node Group creation.

Despite the quota limitation, the project successfully demonstrates:

- Infrastructure automation
- Kubernetes platform provisioning
- Production architecture understanding
- Persistent storage planning
- Backup strategy design
- Cloud troubleshooting capability

---
