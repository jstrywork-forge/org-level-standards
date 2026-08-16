# org-level-standards

The organisation layer of a two-layer AI-assistance model. It holds the rules
that are true of **every** repository, and a pipeline syncs them into each one.

Nothing repo-specific belongs here. The test is one sentence:

> Would this rule still be true in a repository with a different language, a
> different domain and a different team?

If yes, it belongs here. If no, it belongs in that repository's own
`.github/copilot-instructions.md`.

---

## Why two layers

A single hand-maintained instruction file per repository has two failure modes,
and both are expensive:

| Failure | What it looks like |
|---|---|
| **Drift** | Twelve repositories each state the security rule slightly differently. Nobody knows which is current, so nobody trusts any of them |
| **Staleness** | A standard changes. Eleven repositories are updated; the twelfth is found six months later during an audit |

Splitting the layers fixes both. Universal rules have exactly one author and
arrive by pipeline. Local quirks stay local, hand-edited, and small enough that
somebody actually reads them.

**The corollary matters as much as the rule:** because the org layer is synced,
it must never be hand-edited in a consuming repository. An edit there is
overwritten on the next sync, silently, and the person who made it will believe
the standard is being ignored.

## What a consuming repository looks like

```
.github/
├── copilot-instructions.md          ← hand-edited. Repo-specific ONLY.
└── instructions/
    ├── org/                         ← SYNCED from here. Never hand-edit.
    │   ├── angular.instructions.md
    │   ├── dotnet.instructions.md
    │   └── security.instructions.md
    └── org.lock.json                ← which version is pinned, and its hashes
```

## The lock file

Every consuming repository carries one. It records what was synced, from where,
at which version, and the hash of each file:

```json
{
  "source": "https://dev.azure.com/<ORG>/<PROJECT>/_git/engineering-standards",
  "version": "2026.08.1",
  "profile": "dotnet-spa-bff",
  "syncedAt": "2026-08-10T12:00:00Z",
  "files": {
    "angular.instructions.md": "sha256:…",
    "dotnet.instructions.md": "sha256:…",
    "security.instructions.md": "sha256:…"
  }
}
```

**The hashes are the point.** They make three questions answerable without
reading a diff: is this repository on the current standards version, has anyone
hand-edited a synced file, and which repositories still need the sync run. A lock
file without hashes records intent; one with hashes records fact.

### Profiles

A repository takes only the instruction sets its stack needs. `profile` names the
combination, so a pure-Angular repository does not carry .NET rules it will never
apply — noise in an instruction file is not free, it dilutes the rules that
matter.

| Profile | Syncs |
|---|---|
| `spa` | `angular`, `security` |
| `dotnet-api` | `dotnet`, `security` |
| `dotnet-spa-bff` | `angular`, `dotnet`, `security` |

## Enforcement — where the value actually lands

Instructions guide an assistant while code is being written. They do not stop
anything reaching `main`. That is what `azure/pr-review.yml` is for: it runs on
pull request, applies the standards in `azure/pr-review-prompt.md`, and leaves
its findings as comments on the pull request.

The two halves are complementary and neither substitutes for the other:

| Layer | Catches | Timing |
|---|---|---|
| Instructions | Most of it, before the code exists | While writing |
| PR review | What was written anyway | Before merge |

A rule that only exists in an instruction file is advice. A rule that also runs
in the pull request is a standard.

## Adding or changing a rule

1. Change it here, and say in the commit message what problem it prevents. A
   rule whose reason is not recorded is one nobody can argue with later — and
   therefore one nobody can safely remove.
2. Bump `version` using `YYYY.MM.n`.
3. Let the sync pipeline open pull requests against consuming repositories.

**Do not add a rule that the toolchain already enforces.** A lint rule, a
compiler setting or an analyser is better than a sentence, because it fails the
build instead of hoping to be read. Instructions are for what tooling cannot
express.

## Contents

| Path | Holds |
|---|---|
| `instructions/` | The synced instruction sets |
| `azure/pr-review.yml` | The ADO pipeline that reviews pull requests against these standards |
| `azure/pr-review-prompt.md` | What that review is told to look for |
| `docs/` | The reasoning behind the model, for people rather than assistants |
