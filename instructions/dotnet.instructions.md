---
applyTo: '**/*.cs'
standard: dotnet
version: 2026.08.1
---

# .NET — organisation standard

**Synced from `engineering-standards`. Do not edit here.**

## Build settings

These belong in `Directory.Build.props`, set once for the repository:

- `TreatWarningsAsErrors` — a warning nobody is forced to read is decoration.
- `Nullable` and `ImplicitUsings` enabled.
- `EnforceCodeStyleInBuild` with `AnalysisLevel` at `latest-recommended`.

## Analysers

- Suppress a rule in `.editorconfig`, never with an inline pragma, and **always
  with a reason**. A suppression without a reason is indistinguishable from
  giving up.
- Logging goes through `[LoggerMessage]` source-generated partial methods.
  Interpolated logging calls allocate on every call including the ones that are
  filtered out, and CA1873 will fail the build.

## Style

- File-scoped namespaces.
- Braces on every block, including single-statement `if`s.
- `sealed` by default on internal types.
- `async`/`await` throughout; no `.Result` and no `.Wait()`.
- `CancellationToken` on every async public method, and pass it down.

## APIs

- Failures return RFC 9457 Problem Details with a stable machine-readable code.
  The code is what callers switch on; the human-readable text stays free to be
  reworded.
- 400 means the request was malformed. 422 means it was well-formed and refused.
- Validation belongs in the domain, not in the endpoint.

## Testing

- Test behaviour, not implementation.
- Do not mock the thing whose behaviour is the risk. Storage and integration
  concerns are tested against the real dependency.
- **A passing test suite does not prove the application starts.** Every service
  needs at least one test that boots the host.
