# ADR-0013: Customer-managed keys for storage encryption

**Status:** Accepted

## Context
Storage accounts encrypt data at rest by default using Microsoft-managed keys (PMK). Customer-managed keys (CMK) shift the encryption key into a Key Vault under customer control. CMK is the right tool for regulated data and for organizations with key-control requirements.

## Decision
Configure CMK on the portfolio storage account using a 2048-bit RSA key in the Key Vault from module 08. Enable automatic key rotation by leaving the key version field empty.

## Consequences
- Loss of access to the Key Vault key (deletion, accidental purge) makes the storage account data unreadable. Mitigated by purge protection on the KV.
- Key rotation is automatic; storage account picks up new key versions without operator intervention.
- One additional RBAC grant: storage account MI requires `Key Vault Crypto Service Encryption User` on the KV.
