# ADR-0007: Availability Zones for new workloads, AvSet for legacy continuity

**Status:** Accepted

## Context
Availability Sets and Availability Zones both provide HA primitives, with different SLA boundaries. AvSet protects against rack and update-domain failures within a single datacenter. Zones protect against datacenter-level failures within a region.

## Decision
New workloads use Availability Zones. Existing AvSet-based workloads remain on AvSet for continuity unless migration is justified by an HA upgrade.

## Consequences
- Zone-redundant SKUs cost ~10% more than zonal/AvSet SKUs (load balancer Standard, public IP Standard).
- Cross-zone traffic incurs minor data transfer cost.
- The 99.99% VM SLA at Zones level beats AvSet's 99.95%.
