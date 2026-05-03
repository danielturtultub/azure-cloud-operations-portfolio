# ADR-0011: Key Vault RBAC permission mode over access policies

**Status:** Accepted

## Context
Key Vault supports two permission models. Access policies (the legacy model) maintain a per-vault list of identity-to-permission mappings. RBAC mode uses Azure role assignments and inherits the broader Azure RBAC model. Microsoft now recommends RBAC mode as the default.

## Decision
Create new Key Vaults with `--enable-rbac-authorization true`. Use roles like Key Vault Administrator, Secrets User, Crypto User per least-privilege.

## Consequences
- Role granularity is finer than access policies (separate Secrets/Keys/Certificates roles).
- Audit and access patterns align with the rest of Azure RBAC.
- Existing vaults using access policies are migrated only when ownership review is performed; mid-life mode switch is supported but requires planning.
