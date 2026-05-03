# ADR-0009: One centralized Log Analytics workspace vs per-team workspaces

**Status:** Accepted

## Context
Two patterns: one workspace ingesting all subscription telemetry, or per-team workspaces with cross-workspace queries for global views. Centralization simplifies queries; decentralization isolates cost and access.

## Decision
One workspace (`law-portfolio-lab-eastus-01`) for the lab. RBAC at the workspace level controls access. Per-team workspaces would be deployed at production scale where cost and access isolation matter.

## Consequences
- Cross-resource KQL queries are trivial (no `union workspace()`).
- One daily ingestion cap protects total spend.
- Workspace permissions must be carefully managed when team scope grows.
