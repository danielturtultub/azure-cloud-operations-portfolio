# ADR-0003: Management group hierarchy designed; deployed only when access permits

**Status:** Accepted (design-only when single-subscription)

## Context
Management Groups (MGs) provide an inheritance scope above subscriptions for policy and access. A MG hierarchy is the canonical answer for cross-subscription role assignment. The lab uses a single subscription, so a MG hierarchy is not required, but the design influences how role-assignment patterns are described.

## Decision
Document a `Tenant Root MG → Lab MG → Subscription` hierarchy. Deploy `Lab MG` only if Global Administrator access permits the elevate-access toggle. Otherwise, treat the design as the future-state target and use subscription-scope grants in the lab.

## Consequences
- Role-assignment examples reference `User Access Administrator at Lab MG` as the production pattern, with `User Access Administrator at subscription` as the lab equivalent.
- Cross-subscription scenarios are documented even when only one subscription exists.
