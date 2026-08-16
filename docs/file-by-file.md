# File by file

What each file contributes, and what breaks without it. Two sections: the
repository you work in, and the standards repository that feeds it.

---

# Repo level — `whatsapp-template-manager`

## The instruction layer

### `.github/copilot-instructions.md`
**Hand-edited. Repo-specific only.**

The traps and conventions of this codebase — the generated client that must not
be hand-edited, the `ui-kit` seam, the demo layer that must never reach a real
build, why there is no login route.

The test for whether something belongs here: *would this sentence still be true
in a repository with a different language and a different domain?* If yes, it is
an org rule and belongs in the other repo.

Without it, the assistant works from the code alone and re-derives your
conventions badly.

### `.github/instructions/org/*.md`
**Synced. Never hand-edit — the next sync overwrites it.**

Currently `angular.instructions.md` and `security.instructions.md`. Each carries
front matter that decides where it applies:

```yaml
applyTo: '**/*.ts'
```

Editing one here does not change the standard; it changes your copy of it, until
the sync silently reverts it. Propose the change in the standards repo instead.

### `.github/instructions/org.lock.json`
**The receipt.**

```json
{ "version": "2026.08.1", "profile": "spa",
  "files": { "angular.instructions.md": "sha256:c58e13…" } }
```

Three questions become answerable without reading a diff: is this repo on the
current version, has anyone edited a synced file, and which repos still need the
sync. `profile` decides *which* sets sync here — a pure Angular repo takes `spa`
and does not carry .NET rules it will never apply.

Without it, the drift check skips and says the repo is not synced to any version.

### `.github/instructions/pr-review-prompt.md`
**The synced fallback copy of the prompt contract.**

Used only when the standards repo is unreachable. Covered by the same drift
check, so a local edit is reported rather than silently obeyed.

## The gates that already work without any of this

### `.eslintrc.json`
Does what a document cannot: **fails the build**. Two custom rules carry most of
the weight — no direct `@angular/material` import outside `ui-kit`, and no
importing from inside the generated API barrels.

One override is load-bearing and easy to break when moving folders:

```json
{ "files": ["ui-kit/**/*.ts"], "rules": { "no-restricted-imports": "off" } }
```

That is what permits `ui-kit` itself to name Material. A stale path there fails
the library for breaking the seam it exists to provide.

### `angular.json`
Carries the containment that keeps the demo layer out of production:

```json
"fileReplacements": [{ "replace": "src/app/dev/dev-providers.ts",
                       "with": "src/app/dev/dev-providers.noop.ts" }]
```

Plus an assets `ignore` for `dev-data/**`. Nothing outside `src/app/dev/` imports
from it, so the bundler drops the whole folder. **Verify with a build, not a
promise:** `yarn build`, then search `dist/` for `DevBar`.

### `e2e/` and `playwright.config.ts`
Three scenarios that double as the demo. Pacing, headed mode, narration and video
all live in the config behind `DEMO=1` — **the specs are byte-identical in CI and
in a demo**. `e2e/README.md` documents each scenario and, more usefully, lists
what they deliberately do not cover.

## The pipeline

### `azure/pr-review.yml`
Lives here because a pipeline runs from the repository it reviews. The canonical
copy is in the standards repo; this one is the deployed instance.

Read the header comment before changing anything — it lists the three ADO
prerequisites, each of which fails in a way that looks like success.

---

# Org level — `engineering-standards`

## `instructions/*.md`
**The rules that are true everywhere.** One file per stack, plus security.

They are written as corrections rather than tutorials, because that is what an
assistant needs: *"a model trained on public Angular code will suggest the 2020
idiom by default — these are the corrections."*

The security set is the one that earns its keep. It is also the set the review is
told to prioritise.

## `azure/pr-review-prompt.md`
**The instruction given to the reviewer.** A file, not inline YAML, for two
reasons: it is reviewable in a pull request of its own, and it is the portable
half — changing model or vendor edits one `curl`, not this.

Most of its value is restraint. It forbids commenting on anything the build
already enforces, because **a review with forty comments gets skimmed and one
with four gets read**.

## `azure/pr-review.yml`
The canonical pipeline. Consuming repos take a copy.

## `README.md`
Why two layers exist, the lock-file contract, and the profile table.

---

# The one duplication

`azure/pr-review-prompt.md` exists here **and** at
`.github/instructions/pr-review-prompt.md` in the consuming repo. That is
deliberate: the live copy is preferred, the synced copy is the fallback when the
standards repo is unreachable, and the drift check covers both.

Two copies of the same prose is a real risk — this pattern has diverged before
elsewhere in this estate. The mitigation is not discipline, it is that the
mismatch is **reported**.

# What to change, and where

| You want to | Edit |
|---|---|
| Record a trap in this codebase | `copilot-instructions.md` |
| Change a rule for every repo | `instructions/*.md` here, bump `version`, re-sync |
| Change what the reviewer looks for | `azure/pr-review-prompt.md` |
| Make a rule actually fail the build | `.eslintrc.json` — better than any sentence |
| Stop the review running on a path | The branch policy, or a path filter in the pipeline |

**Prefer the fourth row.** A lint rule fails the build instead of hoping to be
read. Instructions are for what tooling cannot express — not a substitute for
tooling that can.
