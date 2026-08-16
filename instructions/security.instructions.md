---
applyTo: '**'
standard: security
version: 2026.08.1
---

# Security — organisation standard

**Synced from `engineering-standards`. Do not edit here; local changes are
overwritten and the drift check will fail the build.**
Propose changes by PR in that repository.

## Secrets

- No secret, key, token, connection string or password in source, YAML, or a
  committed config file. Configuration supplies them at runtime.
- A local-emulator credential published in vendor documentation is not a secret,
  but say so in a comment where it appears, so the next reader does not have to
  work out whether it is one.
- Never log secrets, credentials, tokens or personal data. This includes logging
  a whole request or response object that might contain them.

## Dependencies

- Pin every version. Floating ranges make a build unreproducible.
- A known vulnerability fails the build. To lift a vulnerable transitive
  dependency, add a direct reference at a patched version and comment why.

## Web

- Session identity travels in an httpOnly cookie. No token in browser-accessible
  storage.
- Every state-changing endpoint is CSRF-protected. Where an endpoint is exempt,
  the exemption carries a comment naming what authenticates the caller instead.
- Authorise on the server. A UI guard is a convenience and is never the boundary.
- Never take a tenant, account or user identifier from a request when the session
  already carries it.

## Data

- Parameterise every query.
- Validate at the trust boundary, and validate on write rather than on read.
