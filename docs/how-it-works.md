# How it works

From `git clone` to a comment on a pull request. No diagrams, no layers of
abstraction — just what runs, in order, and what it does when each part is
missing.

---

## The whole thing in one paragraph

Two markdown folders sit in every repository: one you edit, one that arrives by
sync. Your editor's assistant reads both, so most violations never get written.
Lint and the tests catch what they can express. When you open a pull request, a
branch policy fires a pipeline that reads the diff, reads the standards from
their source, asks a model whether the change breaks any of them, and posts one
comment. It never blocks a merge.

## Step by step

### 1. Clone

```bash
git clone https://dev.azure.com/the-js-firm/poc-ai-enablement/_git/whatsapp-template-manager
```

You get both instruction layers with the code. Nothing to install, nothing to
configure, no account to connect.

### 2. Open the folder

`.github/copilot-instructions.md` is picked up automatically by Copilot. The
files in `.github/instructions/org/` carry an `applyTo:` glob in their front
matter, so they attach themselves to matching files:

```yaml
---
applyTo: '**/*.ts'
standard: angular
version: 2026.08.1
---
```

**If you do not use Copilot, these are just readable markdown.** See
[`working-without-ai.md`](working-without-ai.md).

### 3. Write code

Two sources of guidance, and they answer different questions:

| Layer | Answers |
|---|---|
| `copilot-instructions.md` | "What is odd about *this* repository?" |
| `instructions/org/*.md` | "What is true everywhere?" |

Where they conflict, **the repo wins** — but the repo file must say so and give
a reason. A local override nobody wrote down is indistinguishable from someone
not knowing the standard existed.

### 4. Local gates

```bash
yarn start     # runs lint FIRST, then serves
yarn e2e-ci    # three end-to-end scenarios, ~15s
```

A lint failure stops the app starting. That is on purpose: finding it on a build
agent means the feedback arrived several minutes and one context switch too late.

### 5. Push and open a pull request

**The `pr:` block in the pipeline YAML does nothing on Azure Repos.** It only
works for GitHub and Bitbucket. What actually fires the review is a **build
validation branch policy** on `main`, set to optional.

If a PR opens and no build starts, that policy is missing or disabled. That is
the first thing to check, every time.

### 6. The pipeline runs

Four things happen, in this order:

**a. Two checkouts.** The app repo at `fetchDepth: 0` — the diff needs the merge
base, and a shallow clone does not have one — and the standards repo, declared as
a resource:

```yaml
resources:
  repositories:
    - repository: standards
      type: git
      name: engineering-standards
      ref: refs/heads/main
```

That declaration is not decoration. With *Protect access to repositories in YAML
pipelines* enabled — the default on new projects — the job token can only reach
repositories declared this way. A hand-rolled `git clone` fails with `TF401019`
no matter what permissions were granted.

**b. The drift check.** Re-hashes the synced copy in `.github/instructions/org/`
against `org.lock.json`. A mismatch means someone edited a file that the next
sync will overwrite. It warns; it does not fail.

**c. The diff.**

```bash
target="${TARGET_BRANCH#refs/heads/}"
git diff "origin/$target"...
```

`System.PullRequest.TargetBranch` arrives as `refs/heads/main`, not `main`.
Concatenating it straight onto `origin/` produces a revision that does not exist,
and the step swallows the error as an empty diff — a green run that reviewed
nothing.

**d. The review.** Assembles, in order: the prompt contract → this repo's
conventions → the org standards → the diff. One call to Azure OpenAI. One comment
posted back, with the HTTP status checked, because a 403 returns an HTML sign-in
page that a bare `curl` reports as success.

### 7. You decide

Nothing blocks. Fix what is worth fixing, reply to what is wrong. The build gates
the pull request on its own.

## What happens when a part is missing

This is the important table. **Every failure degrades to "no comment", never to a
broken build** — which is what makes it safe to switch on everywhere.

| Missing | Result |
|---|---|
| Standards repo unreachable | Falls back to the synced copy and **says so in the comment footer** |
| No prompt contract | Skips. Logs why |
| No endpoint or key | Skips. Logs why |
| Model returns nothing | Skips |
| Diff over 200KB | Truncates and states in the comment that it did |
| Cannot post the comment | Logs the HTTP code and the exact permission to check |

Read the footer of any comment. It states which standards version was applied and
whether they were fetched live or fell back. **The review tells you what it
judged against**, which is a small thing that buys a lot of trust.

## Six traps, already paid for

Each of these produced a green run and no comment. They are fixed; they are
listed because anyone porting this will hit them again.

| Trap | Symptom |
|---|---|
| `pr:` triggers ignored on Azure Repos | PR opens, no build |
| `TargetBranch` is `refs/heads/main` | "Empty diff; nothing to review" |
| `System.CollectionUri` is already a full URL | Nested URL, clone fails |
| Job auth scope pointed at an identity nobody granted | 403 on the comment POST |
| Protected resources need the repo declared | `TF401019` regardless of ACL |
| A declared resource needs first-use authorization | Run cancels at `Checkpoint.Authorization` before any step |

The last two look identical from the outside and have different fixes. That is
why the pipeline now prints the clone's actual error instead of sending it to
`/dev/null`.
