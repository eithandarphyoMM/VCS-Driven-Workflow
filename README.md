# ☁️ HelloCloud Infrastructure: VCS-Driven Monorepo Workflow

This repository (`VCS-Driven-Workflow`) manages a multi-tier, multi-module AWS cloud infrastructure using **HCP Terraform Remote Execution Workspaces**, **OpenID Connect (OIDC) Dynamic IAM Roles**, and **Cross-Workspace State Sharing**.

---

## 🏗️ 1. Project Architecture & Workspace Mapping

The repository follows a **monorepo design pattern**, where all 6 root modules live in the same codebase but map to separate **HCP Terraform Workspaces** via targeted Working Directories and VCS branches.

| Workspace Name | GitHub Repository | VCS Branch | Working Directory | Description |
| :--- | :--- | :--- | :--- | :--- |
| `backend-security-ws` | `VCS-Driven-Workflow` | `backend-security-branch` | `backend-security` | IAM, Security Groups & Secrets (Backend) |
| `frontend-security-ws` | `VCS-Driven-Workflow` | `frontend-security-branch` | `frontend-security` | IAM, Security Groups & Secrets (Frontend) |
| `backend-network-ws` | `VCS-Driven-Workflow` | `backend-network-branch` | `backend-network` | VPC, Subnets & Route Tables (Backend) |
| `frontend-network-ws` | `VCS-Driven-Workflow` | `frontend-network-branch` | `frontend-network` | VPC, Subnets & VPC Peering (Frontend) |
| `backend-compute-ws` | `VCS-Driven-Workflow` | `backend-compute-branch` | `backend-compute` | EC2 Instances & Cloud-Init User Data |
| `frontend-compute-ws` | `VCS-Driven-Workflow` | `frontend-compute-branch` | `frontend-compute` | EC2 Instances & Cloud-Init User Data |

---

## 🛡️ 2. AWS Authentication via OIDC (Least Privilege Plan/Apply Roles)

To eliminate static access keys, HCP Terraform securely assumes AWS IAM roles via OIDC tokens generated per run phase.

### Required Environment Variables (Organization Variable Set):
Configure these variables under your HCP Terraform Organization (**Settings ➔ Variable sets**):

* **Plan Role (Read-Only Auditing):**
  * `TFC_AWS_PLAN_RUN_ROLE_ARN` = `arn:aws:iam::123456789012:role/HCP-Terraform-Plan-Role` *(Sensitive)*
* **Apply Role (Provisioning Permissions):**
  * `TFC_AWS_APPLY_RUN_ROLE_ARN` = `arn:aws:iam::123456789012:role/HCP-Terraform-Apply-Role` *(Sensitive)*
* **Default Region:**
  * `AWS_REGION` = `us-east-1`

---

## 🔄 3. CI/CD Pipeline & Pull Request Lifecycle

Every change to the infrastructure flows through a strict Gitops pull request lifecycle:

1. **Develop on Branch:**
   Checkout the dedicated module branch (e.g., `frontend-network-branch`), modify code, commit, and push.
   ```bash
   git checkout frontend-network-branch
   git add frontend-network/
   git commit -m "Update VPC peering routes"
   git push origin frontend-network-branch# VCS-Driven-Workflow
