# ADR-0012: System vs user-assigned Managed Identity selection rules

**Status:** Accepted

## Context
Managed Identities come in two flavors. System-assigned MIs are tied to a specific resource and deleted with it. User-assigned MIs are standalone resources that can be attached to many other resources.

## Decision
- Use **system-assigned** MIs when one resource needs an identity that is unique to it (a single VM accessing a Key Vault).
- Use **user-assigned** MIs when multiple resources share the same access pattern (a fleet of VMs all accessing the same Key Vault).

## Consequences
- System-assigned: less management overhead, automatic cleanup, but per-resource RBAC grants.
- User-assigned: one RBAC grant covers all attached resources, but the identity exists independently and must be managed explicitly.
- Federated workload identity (preview) is the future for cross-cloud and on-prem identities, tracked but not yet selected.
