# Architecture

This document describes the topology, naming standard, tagging policy, and high-level design choices for the **Secure Azure Administration Environment** deployed in this portfolio.

## Topology

A hub-and-spoke design with a single subscription in `eastus`. The hub holds shared services (Bastion subnet, future gateway subnet, monitoring egress). Spokes hold workloads.

```mermaid
flowchart LR
    Internet((Internet))

    subgraph Hub["vnet-hub-lab-eastus-01<br/>10.0.0.0/16"]
        direction TB
        SharedSnet[snet-shared<br/>10.0.1.0/24]
        BastionSnet[AzureBastionSubnet<br/>10.0.250.0/26]
        GatewaySnet[GatewaySubnet<br/>10.0.255.0/27]
    end

    subgraph Prod["vnet-spoke-prod-lab-eastus-01<br/>10.1.0.0/16"]
        ProdApp[snet-app<br/>10.1.1.0/24]
        ProdData[snet-data<br/>10.1.2.0/24]
    end

    subgraph Dev["vnet-spoke-dev-lab-eastus-01<br/>10.2.0.0/16"]
        DevApp[snet-app<br/>10.2.1.0/24]
    end

    Internet -.->|Bastion only| BastionSnet
    Hub <-->|peering<br/>allow-gateway-transit| Prod
    Hub <-->|peering<br/>allow-gateway-transit| Dev
    Prod -.x.- Dev
```

Spoke-to-spoke traffic does not transit by default. Spokes communicate via the hub only when the hub hosts an Azure Firewall or NVA. This portfolio documents that pattern in `03-networking/` but does not deploy a Firewall (cost: ~$1.25/hr sustained).

## Address space

| VNet | Range | Purpose |
|---|---|---|
| `vnet-hub-lab-eastus-01` | 10.0.0.0/16 | Shared services, Bastion, future gateway |
| `vnet-spoke-prod-lab-eastus-01` | 10.1.0.0/16 | Production workloads |
| `vnet-spoke-dev-lab-eastus-01` | 10.2.0.0/16 | Dev/test workloads |

Subnet allocations leave room for at least four additional subnets per VNet without re-IPing.

## Resource groups

Resource groups slice by service domain, not by environment. Environment slicing is done via the `Environment` tag.

| Resource group | Contains |
|---|---|
| `rg-network-hub-lab-eastus-01` | All VNets, peerings, NSGs, Bastion |
| `rg-platform-lab-eastus-01` | Log Analytics workspace, Automation Account |
| `rg-identity-lab-eastus-01` | Identity-related shared assets |
| `rg-compute-lab-eastus-01` | VMs and VMSS |
| `rg-storage-lab-eastus-01` | Storage accounts |
| `rg-monitor-lab-eastus-01` | Diagnostic settings, alerts (workspace lives in `rg-platform`) |
| `rg-backup-lab-eastus-01` | Recovery Services Vault |
| `rg-security-lab-eastus-01` | Key Vault, Defender configurations |
| `rg-iac-lab-eastus-01` | Sandbox for Bicep deployments |

The reasoning is captured in `docs/decisions/ADR-0002-tags-vs-rgs.md`.

## Naming standard

```
<type>-<workload>-<env>-<region>-<instance>
```

| Field | Allowed values |
|---|---|
| `type` | rg, vnet, snet, nsg, asg, vm, vmss, kv, st, law, rsv, lb, pip, nic, disk, aa |
| `workload` | hub, spoke-prod, spoke-dev, platform, identity, compute, storage, monitor, backup, security, iac |
| `env` | lab (this portfolio), would be dev/test/prod elsewhere |
| `region` | eastus, with `eus` as abbreviation when name length is constrained |
| `instance` | 01, 02, ... |

Storage accounts and Key Vaults have global name uniqueness requirements; append a 4-character suffix when needed.

## Tagging policy

Every billable resource carries five tags. They are enforced by Azure Policy.

| Tag | Required | Example |
|---|---|---|
| `Environment` | Yes | `lab` |
| `Module` | Yes | `03-networking` |
| `Owner` | Yes | `<your-handle>` |
| `CostCenter` | Yes | `portfolio` |
| `ExpiryDate` | Yes | `2026-06-30` |

`ExpiryDate` is consumed by an Automation runbook in module 09 that flags resources past their expiry. The policy that enforces tag presence is the `lab-hygiene-initiative` defined in module 01.

## Identity layer

Microsoft Entra ID provides the directory. Security groups are used for RBAC role assignments. Microsoft 365 groups are used where group expiration is required. Administrative Units segment delegated user-management for office-shaped scopes. Conditional Access enforces MFA from untrusted locations and requires hybrid-Entra-joined devices for privileged groups (designed; deployed only when a P1 trial is active).

Hybrid identity (on-premises Active Directory synchronized to Entra ID via Microsoft Entra Connect Sync) is documented as a design in module 02 because no on-premises directory exists in the lab.

## Monitoring layer

A single centralized Log Analytics workspace (`law-portfolio-lab-eastus-01`) collects diagnostics from every module. Data Collection Rules (DCRs) and the Azure Monitor Agent (AMA) are the modern path; the deprecated Log Analytics agent is not used. A daily ingestion cap of 0.5 GB protects the budget. Action groups route alerts to email; Logic App and ITSM Connector alternatives are documented in module 06.

## Security layer

A Key Vault in RBAC permission mode (`kv-portfolio-lab-eus-XX`) holds secrets, certificates, and keys. VMs use system-assigned Managed Identity to fetch secrets without storing credentials. Microsoft Defender for Cloud baseline is enabled on the subscription; Defender for Servers P1 is enabled briefly during the JIT demonstration in module 08 then disabled to control cost.

## Decision summary

Architecture decisions are recorded as Architecture Decision Records (ADRs) in [`docs/decisions/`](./docs/decisions/).

| ADR | Decision |
|---|---|
| 0001 | Hub-and-spoke topology over flat single-VNet |
| 0002 | Tags for environment, resource groups for service domain |
| 0003 | Management group hierarchy (designed, deployed only if MG access available) |
| 0004 | Custom policy denying VM SKUs above DS2_v2 |
| 0005 | User Access Administrator scoped to specific resource for delegation, not Owner |
| 0006 | Hub-and-spoke vs flat — detailed cost and complexity tradeoff |
| 0007 | Availability Sets vs Availability Zones — chose Zones for new workloads |
| 0008 | Disable shared key access on storage accounts; enforce Entra ID auth |
| 0009 | Centralized Log Analytics workspace vs per-team workspaces |
| 0010 | Disable soft-delete on Recovery Services Vault for the lab only |
| 0011 | Key Vault RBAC permission mode over access policies |
| 0012 | System-assigned vs user-assigned Managed Identity selection rules |
| 0013 | Customer-managed keys (CMK) vs Microsoft-managed keys for storage |
| 0014 | Bicep over ARM JSON |
| 0015 | OIDC federated credentials over service principal secrets for CI/CD |

Each ADR follows the format: Context, Decision, Consequences. They are short — two pages or less.

## What is deployed vs designed-only

To stay inside the cost target, some Azure services are documented with diagrams and configuration steps but not deployed continuously. The split is:

**Deployed continuously (sustained cost ~$5–10/month):** Log Analytics workspace, one storage account, Key Vault, all VNets/subnets/NSGs/peerings, tagged empty resource groups, Automation Account, action groups, alerts.

**Deployed briefly then torn down (within-session cost ~$1–10):** Bastion, internal Load Balancer with backend VMs, App Gateway v2, VPN Gateway Basic, Recovery Services Vault with one VM backed up, Defender for Servers P1.

**Design-only, no deployment:** Azure Firewall, ExpressRoute, full-scale ASR replication, hybrid identity sync (no on-prem AD available), Front Door, Traffic Manager.

The cost-and-cleanup document tracks every billable resource by lifecycle.
