# Module 00 — Project foundation

The setup module. By the end, you have a versioned GitHub repository, the Azure CLI and PowerShell tooling installed and authenticated, a subscription budget with alert thresholds, the eleven-module folder skeleton committed, and the architecture and tagging standards locked in. No Azure resources are deployed yet beyond an empty resource group, so this module costs nothing.

## What this module demonstrates

| Skill | Where it shows up |
|---|---|
| Source-control discipline | Branch protection, PR-based workflow, conventional commits |
| Documentation as a deliverable | README, architecture doc, ADRs, glossary, naming standard |
| Tooling fluency | Azure CLI, Azure PowerShell, Bicep CLI, Microsoft Graph PowerShell |
| Cost governance from day one | Subscription budget with three alert thresholds before any resource is created |
| Repository hygiene | `.gitignore` blocking private files, MIT license, SECURITY policy |

## Prerequisites

A personal Azure subscription with at least Contributor access. A GitHub account. A workstation with internet access. About two hours of focused time.

## Build steps

This module uses **Azure CLI for tenant operations and Bash for repository scaffolding**. The Portal is used only for the budget configuration where the wizard is faster than the equivalent CLI call.

### 1. Install tooling

```bash
# Azure CLI
az --version || curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# PowerShell 7 + Az + Microsoft Graph modules
pwsh -Command "Install-Module Az -Scope CurrentUser -Force"
pwsh -Command "Install-Module Microsoft.Graph -Scope CurrentUser -Force"

# Bicep
az bicep install
az bicep version
```

Microsoft Graph PowerShell replaces the older `AzureAD` and `MSOnline` modules, both of which are end-of-life. Any script in this portfolio that creates users, invites guests, or reads directory data uses Microsoft Graph.

### 2. Authenticate

```bash
az login --use-device-code
az account show
az account set --subscription "<your-subscription-name>"
```

Confirm the active subscription with `az account show -o jsonc | head -20` before continuing. Every command in this portfolio assumes the working subscription is the right one.

### 3. Create the GitHub repository

```bash
gh repo create secure-azure-operations-portfolio --public \
  --description "Hands-on Azure environment: identity, networking, compute, storage, monitoring, backup, security, and IaC."

git clone https://github.com/<your-handle>/secure-azure-operations-portfolio.git
cd secure-azure-operations-portfolio
```

Configure branch protection on `main` in Settings → Branches: require pull request reviews before merging, require linear history, require conversation resolution. Then create a `dev` branch and work there.

### 4. Scaffold the eleven modules

```bash
for d in 00-project-overview 01-governance-cost 02-identity-rbac 03-networking 04-compute \
         05-storage 06-monitoring-logs 07-backup-recovery 08-security-operations \
         09-iac-automation 10-final-capstone; do
    mkdir -p "$d"/{screenshots,scripts,diagrams,docs}
    echo "# $d" > "$d/README.md"
done

mkdir -p docs/decisions diagrams
```

Commit this skeleton with the message `chore: scaffold eleven module folders`.

### 5. Set the subscription budget

Portal navigation: Subscriptions → your subscription → Cost Management → Budgets → Add. Configure:

- Amount: 100 USD
- Reset period: Monthly
- Expiration: 12 months out
- Alert thresholds: 50%, 80%, 100% of actual cost
- Notification recipients: your email

The full strategy (sustained baseline vs build-and-tear-down vs design-only) is in [`../cost-and-cleanup.md`](../cost-and-cleanup.md).

### 6. Pin the cost dashboard

Cost Analysis → group by Tag → `Module` → Save view → Pin to dashboard. Once resources begin getting tagged in module 01, this view shows per-module spend at a glance. The screenshot of the dashboard is captured later in module 01.

### 7. Initialize the documentation

Create the four top-level docs by copying [`../README.md`](../README.md), [`../architecture.md`](../architecture.md), [`../evidence-and-naming.md`](../evidence-and-naming.md), and [`../cost-and-cleanup.md`](../cost-and-cleanup.md) into your repo root. Adjust subscription names and personal handles. Commit them.

### 8. Add the license, security policy, and gitignore

```bash
# MIT license via gh
gh repo edit --add-topic azure,iam,cloud-operations,bicep,kql,az-administrator
echo "MIT License..." > LICENSE
```

`SECURITY.md` describes that this is a portfolio repository, lab data only, and how to report any concerns. `.gitignore` blocks private files:

```gitignore
# Private working files — never commit
PRIVATE_*
*.env
.env*
*.pem
*.key
*.pfx
secrets/
.terraform/
.bicep/

# OS and editor noise
.DS_Store
Thumbs.db
.vscode/
.idea/
```

### 9. Author the first Architecture Decision Record

Create `docs/decisions/ADR-0001-hub-and-spoke.md` with three sections — Context, Decision, Consequences. The ADR captures *why* the topology is hub-and-spoke rather than flat: shared services centralization, scalable spoke addition, clear policy and routing boundaries, traded against slightly higher initial complexity. Two pages or less.

### 10. Final commit

```bash
git add .
git commit -m "feat(00): project foundation — tooling, scaffolding, governance baseline"
git push origin dev
gh pr create --title "Module 00: project foundation" --body "Foundation set: repo, tooling, scaffolding, budget, license, ADR-0001."
```

## Validation

A reviewer (or future you) should be able to verify:

- `az --version`, `pwsh --version`, and `az bicep version` all return current versions.
- `az account show` shows the intended subscription.
- The eleven module folders exist with `screenshots/`, `scripts/`, `diagrams/`, `docs/` subfolders.
- The repo has a top-level README with the Mermaid architecture diagram rendering on github.com.
- `gh api /repos/<you>/secure-azure-operations-portfolio/branches/main/protection` returns a 200 with required reviews configured.
- Cost Management → Budgets shows `budget-portfolio-monthly` with three alert thresholds.
- `.gitignore` contains `PRIVATE_*` and the file is committed.
- ADR-0001 exists with all three sections filled in.

## Cleanup

Nothing to clean up — no Azure resources were deployed in this module. The budget remains in place for the duration of the portfolio.

**Cost:** $0 spent on this module. Sustained add: $0/month.

## Evidence

| File | Demonstrates |
|---|---|
| `screenshots/00-az-version.png` | CLI tooling installed and authenticated to the working subscription |
| `screenshots/00-budget-three-thresholds.png` | $100/month budget with 50/80/100% alert thresholds configured |
| `screenshots/00-branch-protection.png` | Pull-request requirement on `main` branch |
| `screenshots/00-folder-skeleton.png` | Tree view of the eleven module folders |
| `screenshots/00-readme-renders.png` | Top-level README rendering on github.com with Mermaid diagram visible |
| `diagrams/00-tooling-overview.mmd` | Mermaid showing Az CLI, Az PowerShell, Bicep, Microsoft Graph PowerShell, and where each is used |
| `docs/decisions/ADR-0001-hub-and-spoke.md` | Architecture decision: hub-and-spoke over flat |

## Resume bullets

- Established a production-style Azure portfolio repository with branch-protected main, pull-request workflow, conventional commits, and a published Architecture Decision Record series.
- Configured a $100/month subscription budget with three alert thresholds (50/80/100%) before deploying any billable resource, demonstrating cost governance discipline from day one.
- Standardized tooling on Azure CLI, Azure PowerShell, Microsoft Graph PowerShell, and Bicep CLI, replacing the deprecated AzureRM and AzureAD modules across all automation.
- Authored a tagging policy with five required tags (Environment, Module, Owner, CostCenter, ExpiryDate) and a naming standard enforced by Azure Policy across all subsequent modules.

## Interview story

When asked *"How do you start a new cloud project?"* — the answer is the order of operations in this module. Repository, branch protection, and PR workflow before any code. Tooling and authentication next, with a written record of which modules are deprecated and why. Budget and alerts before the first deployment, because cost surprises are the most preventable failure in cloud work. Tagging and naming standards before the first resource, because retroactive tagging is painful and unreliable. Then, and only then, the first ADR documenting why the topology is what it is. The interviewer is looking for someone who treats foundations as a deliverable, not as overhead — this module is the answer.
