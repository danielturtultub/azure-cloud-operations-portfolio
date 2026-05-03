# ADR-0014: Bicep over ARM JSON

**Status:** Accepted

## Context
ARM JSON is the original ARM template language. Bicep is Microsoft's modern DSL that transpiles to ARM. Bicep adds type checking, modules, terser syntax, and editor tooling, with no functional gap.

## Decision
Author all infrastructure in Bicep. Use `az bicep build` to inspect transpiled ARM when needed, but never author ARM JSON directly.

## Consequences
- Faster authoring, smaller files, fewer errors.
- Bicep CLI is a build dependency for CI/CD.
- All Bicep templates are version-controlled with API versions pinned.
