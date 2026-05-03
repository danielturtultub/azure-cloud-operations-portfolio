# ADR-0010: Disable soft-delete on Recovery Services Vault for the lab only

**Status:** Accepted (lab-specific override)

## Context
Soft-delete protects backup data with a 14-day grace period after stop-protection. In production this is a critical defense against malicious or accidental backup deletion. In the lab, the 14-day period prevents rapid teardown and re-spin cycles.

## Decision
For the lab vault only, disable soft-delete via `az backup vault backup-properties set --soft-delete-feature-state Disable`. Production vaults keep soft-delete enabled. Document this exception explicitly so the override is not copied without thought.

## Consequences
- Lab vaults can be deleted in minutes after stop-protection-with-delete-data.
- The override is a documented exception, not a default. Future production work must re-enable soft-delete.
