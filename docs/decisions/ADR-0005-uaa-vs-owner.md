# ADR-0005: User Access Administrator scoped narrowly over Owner for delegation

**Status:** Accepted

## Context
A team needs the ability to assign roles on a specific resource. The two roles that include `Microsoft.Authorization/roleAssignments/write` plus all other permissions on the target are Owner and User Access Administrator. Granting Owner on the parent RG gives the team far more permission than they need.

## Decision
Grant `User Access Administrator` scoped to the specific resource (a VNet, a Key Vault, a storage account) where the team needs to delegate. Pair with a separate role granting the operational permissions they actually need (Network Contributor, Storage Blob Data Contributor, etc.).

## Consequences
- Two role assignments per delegated team — slightly more setup.
- Blast radius is bounded: a compromised credential cannot grant access to resources outside the assigned scope.
- The pattern transfers to PIM (eligible UAA assignments) when Microsoft Entra ID Premium P2 is available.
