---
applyTo: '**/*.ts'
standard: angular
version: 2026.08.1
---

# Angular — organisation standard

**Synced from `engineering-standards`. Do not edit in a consuming repo.**

## Current-Angular defaults

A model trained on public Angular code will suggest the 2020 idiom by default.
These are the corrections, and they are the most likely thing to get wrong.

- **Standalone components.** No `NgModule` in new code.
- **`@if` / `@for` / `@switch`**, never `*ngIf` / `*ngFor` / `ngSwitch`. `@for`
  requires a `track` expression — omitting it is a compile error, and tracking by
  index defeats the purpose.
- **Signal inputs** — `input()` and `input.required()`, not the `@Input()`
  decorator.
- **`inject()`** over constructor parameter injection.
- **`ChangeDetectionStrategy.OnPush`** on every component.

## State

- Signals for local and shared state. **No state-management library** unless a
  repository documents why its case needs one.
- `computed()` for derived values. A value derived in a getter recalculates on
  every change-detection pass.
- Where a component is zoneless, mutating a plain field will not repaint. State
  that drives the template lives in a signal.

## HTTP

- `HttpClient` through a typed service. Components do not call HTTP directly.
- **Translate transport errors into domain-meaningful messages** in that service.
  A raw `HttpErrorResponse` surfaced to a user reads
  `Http failure response for /api/x: 422 Unprocessable Content` while the
  response body beside it names the actual rule that was broken.
- Where the API is same-origin behind a proxy, request paths are relative. No
  base URL, no environment file holding one.

## Component libraries

Where a repository wraps a third-party component library behind its own kit,
**feature code imports only from that kit.** The wrapper is what makes replacing
the library a change to one project instead of to every screen, and it stops
being that the moment one feature imports the underlying library directly.

## Accessibility

- Never carry meaning by colour alone. A status colour needs a label beside it.
- Interactive elements are reachable and operable by keyboard.
- Form controls have associated labels — a placeholder is not a label.

## Testing

- Test behaviour through the component's public surface, not its internals.
- Prefer a real dependency where it is cheap; mock only what is slow or
  non-deterministic.
- A component that cannot be tested without mocking six things is telling you
  about its design, not about the test framework.
