# ADR-0015: OIDC federated credentials over service principal secrets for CI/CD

**Status:** Accepted

## Context
GitHub Actions can authenticate to Azure via a service principal client secret stored as a GitHub Secret, or via OIDC federation where GitHub presents a token validated by Microsoft Entra ID against a configured federated credential.

## Decision
All CI/CD workflows authenticate via OIDC federation. The federated credential's `subject` field scopes the trust to a specific repo and branch (`repo:<org>/<repo>:ref:refs/heads/main`). No client secret is stored.

## Consequences
- No secret to rotate, no secret to leak, no secret to copy across environments.
- The federation subject is the security boundary — only workflows running on the named branch of the named repo can authenticate.
- Pull requests from forks cannot impersonate the credential.
- Each environment (production, staging) gets its own federated credential to avoid cross-environment authentication via PR.
