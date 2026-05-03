# ADR-0006: Hub-and-spoke topology — detailed cost and complexity tradeoff

**Status:** Accepted

## Context
This ADR extends ADR-0001 with the cost-and-complexity comparison.

## Decision
Hub-and-spoke pays for itself starting at three workloads sharing common services. Below three, a flat VNet is simpler. The portfolio has two spokes plus design-room for more, so hub-and-spoke is the right starting point.

## Consequences
- Operational complexity: ~1.5x flat VNet (peerings, gateway-transit flag pair).
- Cost: peerings are free; data transfer charges apply at scale but are negligible at lab volumes.
- Future migration cost from flat to hub-and-spoke is high (re-IPing spokes); starting hub-and-spoke avoids that future cost.
