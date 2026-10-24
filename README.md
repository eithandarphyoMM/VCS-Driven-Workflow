# ☁️ HelloCloud Infrastructure: VCS-Driven Monorepo Workflow

This repository (`VCS-Driven-Workflow`) manages a multi-tier, multi-module AWS cloud infrastructure using **HCP Terraform Remote Execution Workspaces**, **OpenID Connect (OIDC) Dynamic IAM Roles**, and **Cross-Workspace State Sharing**.

---

## 🏗️ 1. Project Architecture & Workspace Mapping

The repository follows a **monorepo design pattern**, where all root modules live in the same codebase but map to separate **HCP Terraform Workspaces** via targeted Working Directories and glob patterns (`path/**/*`).

| Workspace Name | GitHub Repository | VCS Branch | Working Directory | Glob Path Pattern | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `frontend-security-ws` | `VCS-Driven-Workflow` | `main` | `frontend-security` | `frontend-security/**/*` | IAM & Security Groups (Frontend) |
| `backend-security-ws` | `VCS-Driven-Workflow` | `main` | `backend-security` | `backend-security/**/*` | IAM & Security Groups (Backend) |
| `backend-network-ws` | `VCS-Driven-Workflow` | `main` | `backend-network` | `backend-network/**/*` | VPC, Subnets & Route Tables (Backend) |
| `frontend-network-ws` | `VCS-Driven-Workflow` | `main` | `frontend-network` | `frontend-network/**/*` | VPC, Subnets & VPC Peering (Frontend) |
| `backend-compute-ws` | `VCS-Driven-Workflow` | `main` | `backend-compute` | `backend-compute/**/*` | EC2 Instances & Cloud-Init User Data |
| `frontend-compute-ws` | `VCS-Driven-Workflow` | `main` | `frontend-compute` | `frontend-compute/**/*` | EC2 Instances & Cloud-Init User Data |

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
   Checkout a feature branch, modify code inside a specific folder (e.g., `frontend-security/`), commit, and push.
   ```bash
   git checkout -b feature/update-frontend-security
   git add frontend-security/
   git commit -m "Update frontend security rules"
   git push origin feature/update-frontend-security