# Evidence and naming

This document defines how screenshots, diagrams, and configuration evidence are captured and stored across the portfolio. The goal is that anyone reviewing the repo — recruiter, hiring manager, peer engineer — can verify that the work is real, current, and reproducible.

## Folder layout per module

Every module folder has the same subfolders:

```
NN-module-name/
├── README.md             ← the module narrative, build steps, and evidence index
├── screenshots/          ← PNGs proving each configuration was real
├── diagrams/             ← .mmd Mermaid sources; PNG exports if needed
├── scripts/              ← deployable code: Bicep, Bash, PowerShell, KQL queries
└── docs/                 ← supporting markdown: ADRs scoped to the module, design docs
```

The README references files in these subfolders by exact filename so a reader can click through.

## Screenshot filename convention

```
NN-<topic>-<state>.png
```

Where:

- `NN` is the module number (`02-` for identity, `06-` for monitoring, etc.).
- `<topic>` is short kebab-case describing what the screenshot shows.
- `<state>` indicates configuration phase when relevant: `before`, `after`, `success`, `failure`, `denied`.

### Examples

| Filename | What it shows |
|---|---|
| `02-bulk-create-members.png` | Bulk-create wizard completed for member users |
| `02-contributor-cannot-assign.png` | Role-assignment denial for a user with only Contributor permissions |
| `03-nsg-service-tag-keyvault.png` | NSG rule using the AzureKeyVault service tag |
| `03-peering-hub-prod-flags.png` | Peering configuration showing gateway-transit flags |
| `06-kql-event-search-error.png` | KQL query results for `Event \| search "error"` |
| `07-vm-full-restore-new-vm.png` | Recovery Services Vault restore wizard with "Create new" target |
| `08-mi-fetches-secret.png` | VM using Managed Identity to fetch a Key Vault secret |
| `09-rg-deployments-blade.png` | Resource group Deployments blade showing template history |

## Diagram filename convention

```
NN-<topic>.mmd            (Mermaid source)
NN-<topic>.png            (rendered export, optional)
```

### Examples

| Filename | Diagram |
|---|---|
| `03-hub-spoke-gateway-transit.mmd` | Hub-and-spoke peering with gateway-transit flag flow |
| `03-ilb-sql-listener.mmd` | Internal Load Balancer fronting SQL Always On listener |
| `05-azure-files-identity-auth.mmd` | Azure Files identity-based authentication paths |
| `06-azmon-architecture.mmd` | Azure Monitor data plane: agents, DCRs, workspace, alerts |
| `07-asr-hyperv-architecture.mmd` | Azure Site Recovery for Hyper-V virtual machines |
| `09-cicd-flow.mmd` | GitHub Actions pipeline: what-if, plan, deploy with OIDC |

Mermaid sources are preferred because they render directly on GitHub and are diff-friendly. PNG exports are added only when a diagram needs to render in a context that does not support Mermaid (LinkedIn post, slide deck).

## Script filename convention

Scripts are named for what they do, not for the question they came from.

```
scripts/
├── modules/
│   ├── vnet.bicep
│   ├── nsg.bicep
│   ├── storage.bicep
│   └── keyvault.bicep
├── queries/
│   ├── event-error.kql
│   ├── kv-unknown-caller.kql
│   └── nsg-deny-trace.kql
├── bulk-guest-invite.ps1
├── runbook-vm-schedule.ps1
└── cleanup-orphans.sh
```

## Redaction rules

Before any screenshot or output is committed to the public repo, these elements are blurred or removed:

- Subscription IDs (full GUID)
- Tenant IDs (full GUID)
- Object IDs of users, groups, service principals
- Personal email addresses (use generic `user@contoso-test.com` or blur)
- Resource IDs that contain a real subscription GUID
- IP addresses that map to your home or workplace
- Any actual secret value, even base64-encoded
- API keys, access keys, SAS tokens, connection strings

Sometimes a screenshot is more useful with a partial GUID visible — for example, showing that a Managed Identity object ID matches a role assignment scope. In that case, blur the last half of each GUID and keep the first six characters; that is enough to demonstrate the match without exposing the actual identifier.

## Evidence index per module

Each module README ends with an evidence table that lists every screenshot and diagram it references. The reviewer can scan this table to check that the claimed evidence exists.

```markdown
## Evidence

| File | Demonstrates |
|---|---|
| `screenshots/03-peering-hub-prod-flags.png` | Hub-to-spoke peering created with allow-gateway-transit |
| `screenshots/03-peering-prod-hub-flags.png` | Spoke-to-hub peering created with use-remote-gateways |
| `diagrams/03-hub-spoke-gateway-transit.mmd` | Topology showing the bidirectional flag pair |
| `scripts/modules/vnet.bicep` | Reusable Bicep module producing the hub VNet |
```

## Validation evidence beyond screenshots

A screenshot proves a configuration existed. To prove the configuration *worked*, modules also include:

- **CLI output** captured to a `.txt` file in `scripts/` (with sensitive values redacted).
- **KQL query results** exported as CSV when the result count or aggregation matters.
- **Error captures** showing intentional misconfigurations being denied — for example, a VM deployment in a non-allowed location returning a policy denial.
- **Before/after pairs** when the work is a remediation: the failing state, then the corrected state.

## Reproducing the evidence

Each module README includes a "How to reproduce" section listing the exact commands. A peer running through these commands in their own subscription should produce screenshots that look essentially identical, modulo their own subscription IDs, names, and timestamps. This is the test for whether the evidence is genuine: someone else can recreate it.

## Common evidence mistakes to avoid

A few patterns weaken the portfolio when reviewers see them:

Screenshots that show only the result without showing the configuration that produced it. Always pair "what I configured" with "what it did".

Screenshots cropped so tightly that the resource name, region, or scope is invisible. The viewer cannot tell whether this is the resource you describe.

Diagrams without labels — generic "VNet → VM" boxes. Use real names from the naming standard.

Mass screenshots from a single five-minute session. Real operational work has timestamps spread over days; a portfolio that shows all timestamps within ten minutes looks staged. Spread the build and screenshots over multiple sessions.

Screenshots of the Azure Portal home page or a navigation breadcrumb with nothing meaningful in frame. Every screenshot should be evidence of a specific decision or result.

Pixelated or resized screenshots that hide details. Capture at native resolution; PNG, not JPEG.
