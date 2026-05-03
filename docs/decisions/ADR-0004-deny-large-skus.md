# ADR-0004: Custom policy denying VM SKUs above DS2_v2 baseline

**Status:** Accepted

## Context
The lab budget is $100/month. A single misconfigured VM at Standard_E32s_v3 (~$1,000/month) would exceed the budget many times over within hours. Built-in policies do not include a "max-spend SKU" rule.

## Decision
Author a custom Azure Policy with effect=deny on `Microsoft.Compute/virtualMachines/sku.name` not in `[Standard_B1s, Standard_B2s, Standard_DS1_v2, Standard_DS2_v2]`. Assign at subscription scope as part of the Lab Hygiene Initiative.

## Consequences
- Accidental large-SKU deployments fail at the ARM API, before any cost is incurred.
- New SKU approvals require updating the policy parameter list — a deliberate friction that protects the budget.
- Production environments expand the allowlist; the policy mechanism transfers cleanly.
