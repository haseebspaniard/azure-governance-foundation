# Azure Project 20 — Governance & Cost Management Foundation

> **Part of my Cloud & DevOps learning journey** | [Medium](https://medium.com/@haseebabdul480) · [LinkedIn](https://www.linkedin.com/in/abdulhaseebas) · [GitHub](https://github.com/haseebspaniard)

---

## Overview

This project sets up the **governance and cost management foundation** that real enterprises establish *before* deploying any workloads on Azure. It covers the full organizational hierarchy, resource tagging, custom access control, and budget alerting — the scaffolding every cloud environment needs.

> In a real company, nobody launches a single VM before this foundation is in place. This project taught me why.

---

## Architecture

```
Tenant Root Group
 └── Haseeb DevOps Learning (Management Group)
        └── Azure Subscription 1
               ├── rg-network   [Environment=Dev | Owner=Haseeb | CostCenter=IT]
               ├── rg-compute   [Environment=Dev | Owner=Haseeb | CostCenter=IT]
               └── rg-storage   [Environment=Dev | Owner=Haseeb | CostCenter=IT]
```

---

## Tasks Completed

| # | Task | Status |
|---|------|--------|
| 1 | Create Management Group hierarchy | ✅ Done |
| 2 | Create 3 Resource Groups (network/compute/storage) | ✅ Done |
| 3 | Apply tags (Environment, Owner, CostCenter) to all RGs | ✅ Done |
| 4 | Create custom RBAC role "VM Operator" (start/stop/restart — no delete) | ✅ Done |
| 5 | Assign "VM Operator" role at subscription scope | ✅ Done |
| 6 | Set up $5 monthly budget alert (80% + 100% thresholds) | ✅ Done |

---

## Key Commands Used

```bash
# 1. Create Management Group
az account management-group create \
  --name mg-haseeb-devops \
  --display-name "Haseeb DevOps Learning"

# 2. Move subscription into Management Group
az account management-group subscription add \
  --name mg-haseeb-devops \
  --subscription <SUBSCRIPTION-ID>

# 3. Create Resource Groups
az group create --name rg-network --location centralindia
az group create --name rg-compute --location centralindia
az group create --name rg-storage --location centralindia

# 4. Apply tags to all Resource Groups
az group update --name rg-network \
  --set tags.Environment=Dev tags.Owner=Haseeb tags.CostCenter=IT

az group update --name rg-compute \
  --set tags.Environment=Dev tags.Owner=Haseeb tags.CostCenter=IT

az group update --name rg-storage \
  --set tags.Environment=Dev tags.Owner=Haseeb tags.CostCenter=IT

# 5. Verify tags
az group show --name rg-network --query tags

# 6. Create custom RBAC role from JSON definition
az role definition create --role-definition vm-operator-role.json

# 7. Assign role
az role assignment create \
  --assignee "<user-email>" \
  --role "VM Operator" \
  --scope "/subscriptions/<SUBSCRIPTION-ID>"

# 8. Verify role assignment
az role assignment list \
  --assignee "<user-email>" \
  --query "[?roleDefinitionName=='VM Operator']" \
  -o table

# 9. List all custom roles (verification)
az role definition list --custom-role-only true --query "[].roleName" -o table
```

---

## Custom RBAC Role Definition

```json
{
  "Name": "VM Operator",
  "IsCustom": true,
  "Description": "Can start, stop, and restart VMs but cannot delete them.",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<SUBSCRIPTION-ID>"
  ]
}
```

### Why this role matters
Azure RBAC uses an **explicit allow** model — if an action isn't listed in `Actions`, it's denied by default. By omitting `Microsoft.Compute/virtualMachines/delete`, delete access is automatically blocked. This is the **Principle of Least Privilege** applied to cloud access control.

---

## What I Learned

- **Management Groups** allow centralized policy and RBAC inheritance across multiple subscriptions — essential for enterprise-scale governance
- **Resource Groups** should be organized by lifecycle and team ownership, not just by project
- **Tags** are the backbone of cost tracking and resource accountability at scale — without them, cloud billing becomes impossible to analyze
- **Custom RBAC roles** are how enterprises apply Principle of Least Privilege precisely — built-in roles are often too broad
- **Budget alerts** are a first line of defence against surprise cloud bills — every subscription should have one before any resources are deployed

---

## Challenges & Notes

- `az consumption budget create` CLI returned a 400 error (known API version incompatibility in preview command group) — budget was created via Azure Portal UI instead, which is common practice in real environments
- Institutional tenant (akumenbyq.com) restricted test user creation — role assignment was demonstrated on the primary account to show the mechanism
- `!` in passwords inside double quotes triggers Bash history expansion — fixed by using single quotes around password strings

---

## Tech Used

![Azure](https://img.shields.io/badge/Microsoft_Azure-0089D6?style=flat&logo=microsoft-azure&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Cloud Shell](https://img.shields.io/badge/Azure_Cloud_Shell-0089D6?style=flat&logo=microsoft-azure&logoColor=white)

---

## Part of a Larger Series

This project is **Project 20** in my structured Cloud & DevOps training. The foundation built here (Management Groups, Resource Groups, tagging) is used in all subsequent Azure projects.

| Project | Topic |
|---------|-------|
| Project 20 | Governance & Cost Management Foundation ← You are here |
| Project 21 | Networking Backbone (VNet, NSGs, Bastion) |
| Project 22+ | Coming soon... |

---

*Built by Abdul Haseeb AS — Former ICT Teacher transitioning into Cloud & DevOps*
*[medium.com/@haseebabdul480](https://medium.com/@haseebabdul480) · [linkedin.com/in/abdulhaseebas](https://www.linkedin.com/in/abdulhaseebas)*
