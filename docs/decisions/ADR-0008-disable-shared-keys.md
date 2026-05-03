# ADR-0008: Disable shared key access on storage accounts

**Status:** Accepted

## Context
Storage accounts support two authentication models: shared key (account key, derived SAS tokens) and Microsoft Entra ID (RBAC roles). Shared keys are full-access credentials with no per-identity attribution. Compromise requires full key rotation.

## Decision
Set `--allow-shared-key-access false` at storage account creation. All data-plane access goes through Microsoft Entra ID with role grants. AzCopy, applications, and CI use user-delegation SAS or direct RBAC.

## Consequences
- Every caller needs an explicit RBAC grant — initial setup friction.
- Audit trails identify the calling principal per operation.
- Compromised credentials are revoked per-identity, not per-account.
- Tools that only support shared keys (rare in 2026) cannot be used.
