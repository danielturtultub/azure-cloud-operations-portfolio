# Cost and cleanup

This portfolio runs on a personal subscription with a $100/month budget cap. Sustained baseline cost is $5–10/month; the rest of the budget is reserved for short bursts when expensive services are deployed, screenshotted, validated, and torn down inside the same session. This document catalogs every billable resource, its lifecycle policy, the cleanup commands to run when work is done, and the daily orphan-resource audit that catches anything missed.

## Budget configuration

A monthly subscription budget is set at $100 with action-group alerts at three thresholds.

```bash
az consumption budget create \
  --budget-name budget-portfolio-monthly \
  --amount 100 \
  --category Cost \
  --time-grain Monthly \
  --start-date 2026-01-01 \
  --end-date 2026-12-31
```

Notification thresholds: 50% (early warning), 80% (decision point — pause non-essential builds), 100% (kill-switch — tear down all non-baseline resources). Alerts route to an action group named `ag-portfolio-email` that emails the owner.

The budget is also pinned to a custom Azure dashboard alongside Cost Analysis grouped by the `Module` tag, so per-module spend is visible at a glance.

## Sustained baseline (always on)

These resources stay deployed across the life of the portfolio. Estimated combined cost: $5–10/month.

| Resource | Type | Estimated monthly cost | Notes |
|---|---|---|---|
| Resource groups (9) | RG | $0 | Empty RGs are free |
| `vnet-hub-lab-eastus-01` and 2 spokes | VNet | $0 | VNets and peerings are free; data transfer billed at lab volumes is negligible |
| Network Security Groups | NSG | $0 | Rule evaluation is free |
| `law-portfolio-lab-eastus-01` | Log Analytics workspace | $1–4 | Pay-as-you-go ingestion, capped at 0.5 GB/day |
| `kv-portfolio-lab-eus-XX` | Key Vault | <$0.10 | Operations billed; minimal at lab use |
| `stportfoliolabeusXX` | Storage account | <$1 | Standard LRS, GPv2, low data volume |
| `aa-portfolio-lab-eus-01` | Automation Account | $0 | First 500 minutes/month free |
| Action group `ag-portfolio-email` | Azure Monitor | $0 | Email actions are free |
| 1–3 metric alerts | Azure Monitor | <$0.50 | $0.10 per alert per month |
| 1 log search alert | Azure Monitor | $1.50 | Per Microsoft pricing for log alerts |
| Tags applied across all | — | $0 | |

A sustained baseline this lean is achievable because the portfolio uses **build → screenshot → tear down** for everything expensive.

## Build-then-tear-down catalog

These resources are deployed for a specific demonstration, screenshotted with all relevant evidence captured, validated, then deleted before the session ends. Each entry includes the within-session cost and the cleanup commands.

### Azure Bastion

**Why it's worth deploying briefly:** Modern remote-access pattern that eliminates public IPs on VMs. Strong interview talking point. Used in the compute and security modules to demonstrate browser-based SSH/RDP.

**Within-session cost:** ~$5/day, ~$140/month if left running.

**Lifecycle:** Deploy at the start of a Bastion-demonstration session, capture connection screenshots, delete before logging off.

```bash
# Deploy
az network public-ip create -g rg-network-hub-lab-eastus-01 -n pip-bastion-lab-eastus-01 --sku Standard --allocation-method Static
az network bastion create -g rg-network-hub-lab-eastus-01 -n bastion-hub-lab-eastus-01 \
  --public-ip-address pip-bastion-lab-eastus-01 --vnet-name vnet-hub-lab-eastus-01 --location eastus --sku Basic

# Tear down
az network bastion delete -g rg-network-hub-lab-eastus-01 -n bastion-hub-lab-eastus-01
az network public-ip delete -g rg-network-hub-lab-eastus-01 -n pip-bastion-lab-eastus-01
```

### Application Gateway v2 (optional)

**Why it's worth deploying briefly:** WAF and L7 routing are recurring interview topics. Even a single deployment + screenshot of path-based routing rules is meaningful evidence.

**Within-session cost:** ~$0.20/hr plus capacity units; figure $5–10 for a half-day demonstration.

**Lifecycle:** Deploy in module 03 only if you can finish the demonstration in one session. Otherwise, leave as design-only with a Mermaid diagram.

### VPN Gateway Basic

**Within-session cost:** ~$0.04/hr (~$1/day). Affordable for a one-day demonstration.

**Lifecycle:** Deploy briefly for a single screenshot of the GatewaySubnet populated with a VPN Gateway resource. Tear down same day.

### Internal Load Balancer with backend VMs

**Within-session cost:** Standard LB ~$0.025/hr; backend VMs (2× B1s) ~$0.024/hr. Roughly $0.50/day.

**Lifecycle:** Deploy in module 03 to demonstrate health probes and load-balancing rules. Tear down within 24 hours.

```bash
# Tear down
az network lb delete -g rg-network-hub-lab-eastus-01 -n ilb-app-lab-eastus-01
az vm delete -g rg-compute-lab-eastus-01 -n vm-app-lab-eus-01 --yes
az vm delete -g rg-compute-lab-eastus-01 -n vm-app-lab-eus-02 --yes
# Then run the orphan disk and NIC cleanup below
```

### Recovery Services Vault with one VM backed up

**Within-session cost:** Vault is free; first backup ~$10/month per protected instance plus storage. Soft-delete adds 14 days of retention cost after disable.

**Lifecycle:** Deploy with soft-delete disabled for the lab so cleanup is fast. Back up one VM, demonstrate file-level and full-VM restore, stop protection with delete-data, then delete the vault.

```bash
# Stop protection and delete data
az backup protection disable --vault-name rsv-portfolio-lab-eastus-01 \
  --resource-group rg-backup-lab-eastus-01 \
  --container-name <container> --item-name <item> \
  --delete-backup-data true --yes
# Then delete the vault
az backup vault delete --name rsv-portfolio-lab-eastus-01 --resource-group rg-backup-lab-eastus-01
```

Soft-delete should remain enabled in production. The lab disables it only because the goal is short cycle time, and the documented decision is captured in `docs/decisions/ADR-0010-rsv-soft-delete-lab.md`.

### Defender for Servers P1

**Within-session cost:** ~$15/server/month after the trial ends. A free 30-day trial covers initial demonstrations.

**Lifecycle:** Enable for the JIT demonstration in module 08. Capture screenshots of secure score before/after. Disable the plan immediately after.

```bash
# Disable Defender for Servers
az security pricing create -n VirtualMachines --tier Free
```

### Test VMs for ad-hoc demonstrations

**Within-session cost:** Standard_B1s ~$0.012/hr (~$0.30/day). Negligible if torn down same session.

**Lifecycle:** Most demonstrations need only 1–2 hours. Always combine `az vm delete` with disk and NIC cleanup, since these do not auto-delete in older API versions.

```bash
# VM cleanup with disk and NIC sweep
VM=vm-test-lab-eus-01
RG=rg-compute-lab-eastus-01
az vm delete -g $RG -n $VM --yes
# Sweep dependents
az disk list -g $RG --query "[?managedBy==null].id" -o tsv | xargs -r -n1 az disk delete --yes --ids
az network nic list -g $RG --query "[?virtualMachine==null].id" -o tsv | xargs -r -n1 az network nic delete --ids
```

## Document-only (never deployed)

These services have high deployment cost relative to lab demonstration value. They are documented with diagrams, configuration walkthroughs, and decision records, but not deployed.

| Service | Reason for design-only |
|---|---|
| Azure Firewall Standard | ~$1.25/hr (~$900/month). Cost dominates everything else. |
| ExpressRoute | Requires a partner circuit; inappropriate for solo lab. |
| Full ASR replication | Per-VM replication storage and compute add up; the architecture diagram alone demonstrates the concept. |
| Hybrid identity sync (Entra Connect) | No on-premises Active Directory available in the lab. Documented with diagram and the modern Cloud Sync alternative. |
| Front Door | Sustained ~$35/month base; not necessary to demonstrate the concept. |
| Traffic Manager | Per-DNS-query and per-endpoint billing; design and screenshot of profile types is sufficient. |
| Azure Synapse, HDInsight, Databricks | Out of scope for an Azure administration portfolio. |

## Daily orphan-resource audit

Untracked resources are the primary cause of unexpected lab spend. A bash script in `09-iac-automation/scripts/cleanup-orphans.sh` catches the four most common offenders.

```bash
#!/usr/bin/env bash
# Lists orphan resources that bill silently. Review before deleting.

echo "=== Unattached managed disks ==="
az disk list --query "[?managedBy==null].[name,resourceGroup,diskSizeGb,sku.name]" -o table

echo "=== Unattached public IPs ==="
az network public-ip list --query "[?ipConfiguration==null].[name,resourceGroup,sku.name]" -o table

echo "=== Unattached network interfaces ==="
az network nic list --query "[?virtualMachine==null].[name,resourceGroup]" -o table

echo "=== Resources past their ExpiryDate tag ==="
TODAY=$(date +%Y-%m-%d)
az resource list --query "[?tags.ExpiryDate != null && tags.ExpiryDate < '$TODAY'].[name,type,resourceGroup,tags.ExpiryDate]" -o table
```

The script is run from a local workstation, not as a runbook, because the goal is human review before deletion. Anything stale is either deleted or has its `ExpiryDate` extended with a documented reason.

## End-of-portfolio teardown

When you decide the portfolio is done — exam passed, job landed, moved on — the full teardown is one command per resource group, in dependency order.

```bash
# Order matters: delete dependent RGs before the ones that hold shared services
for rg in rg-iac-lab-eastus-01 rg-compute-lab-eastus-01 rg-storage-lab-eastus-01 \
          rg-monitor-lab-eastus-01 rg-backup-lab-eastus-01 rg-security-lab-eastus-01 \
          rg-network-hub-lab-eastus-01 rg-identity-lab-eastus-01 rg-platform-lab-eastus-01; do
  echo "Deleting $rg"
  az group delete --name $rg --yes --no-wait
done

# Wait, then verify
sleep 600
az group list -o table | grep lab
```

If a resource group refuses to delete due to a lock, list and remove the lock first:

```bash
az lock list -g <rg> -o table
az lock delete --name <lock-name> -g <rg>
```

If a deletion fails on a Recovery Services Vault with protected items, stop protection with delete-data first (see the RSV section above).

## Cost reflection per module

Each module README ends with a cost summary line in this format:

```
**Cost:** $X spent on this module. Sustained add: $Y/month.
```

These numbers feed into a single capstone roll-up in `10-final-capstone/README.md` so the total portfolio cost can be reviewed against the $100/month budget.
