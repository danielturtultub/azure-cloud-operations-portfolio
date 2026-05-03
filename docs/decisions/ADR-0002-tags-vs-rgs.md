# ADR-0002: Tags for environment, resource groups for service domain

**Status:** Accepted

## Context
Resource groups can slice resources by environment (`rg-prod`, `rg-dev`) or by service domain (`rg-network`, `rg-compute`). Choosing one fixes the operational pattern for the portfolio's lifetime. Mixing both produces hybrid groupings that confuse cost reports and access boundaries.

## Decision
Resource groups slice by service domain. Environment is expressed as the `Environment` tag (`lab`, eventually `dev`/`test`/`prod`). All cost reports group by `Module` tag, not by RG.

## Consequences
- Cost views remain stable as environments scale.
- An RBAC grant at RG scope grants service-domain authority, which matches typical team structure (network team, compute team).
- Per-environment access boundaries require tag-based access conditions or a Lab MG hierarchy, both designed in module 02.
