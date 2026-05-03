# ADR-0001: Hub-and-spoke topology over flat single-VNet

**Status:** Accepted

## Context
The portfolio needs a network topology that supports multiple workloads, centralized identity and monitoring services, and a clear path to add VPN or Firewall services later without rearchitecting. A flat single-VNet design is simpler initially but couples all workloads to one address space, one routing table, and one set of NSGs.

## Decision
Use a hub-and-spoke topology with one hub VNet (`vnet-hub-lab-eastus-01`) and two workload spokes (`vnet-spoke-prod`, `vnet-spoke-dev`). Hub holds shared services and reserved subnets for future gateway and Bastion deployments. Spokes connect to the hub via VNet peering with `allow-gateway-transit` on the hub side and `use-remote-gateways` on the spoke side, enabling future centralized gateway placement.

## Consequences
- New workloads land in new spokes without touching existing spokes.
- Centralized services (DNS, monitoring egress, Bastion) live in one place.
- Spoke-to-spoke traffic does not transit by default — requires Azure Firewall or NVA in the hub. Documented as design-only because of cost.
- Slightly more peering relationships to manage; mitigated by a custom RBAC role (VNet Peering Manager).
