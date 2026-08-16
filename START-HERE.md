# Start here

You have just cloned this. Read this file, then decide what to do with your day.
It is the only document that goes stale, so it says the date it was written.

**Written 2026-08-16, late evening.**

---

## What exists, and what it does

Two repositories in `the-js-firm/poc-ai-enablement`:

| Repo | Job |
|---|---|
| `whatsapp-template-manager` | An Angular app. Runs with `yarn start`, no backend. Also the demo |
| `engineering-standards` | **This one.** The rules every repo shares, and the pipeline that applies them |

The claim being tested is narrow and worth stating plainly:

> Lint, the compiler and the tests catch what they can express. There is a class
> of mistake they cannot express — a component reaching past the layer that
> translates errors, a tenant id taken from a URL, a status shown only as a
> colour. Those need a reviewer who has read the standards.

Two pull requests in the app repo prove it. Both contain defects. Both passed
lint, both builds and all three end-to-end tests. The only thing that had
anything to say about them was the standards layer.

## Where it stands

| | State |
|---|---|
| Both repos on `main` | Done |
| PR review pipeline | Working — comments on PRs #1 and #2 |
| Standards fetched live at review time | Working |
| Drift check against the lock file | Working |
| Azure OpenAI `gpt-4o` in a personal subscription | Working |
| `dry-run` branch, ready, deliberately without a PR | Ready |
| **A full rehearsal, start to finish** | **Not done** |

## The gaps, honestly

**1. Nobody has rehearsed it.** Everything is staged. Six bugs surfaced while
building this, and every one of them produced a *green run with no comment* —
the failure that looks like success. Walk it once before showing anyone.

**2. The review missed the most serious defect.** On PR #2 it flagged a
`console.log` of the session but not the fact that a tenant identifier was being
read from the URL. Say this before someone finds it. It is a second pair of
eyes, not a gate, and the pipeline says so in its own header.

**3. The sync is manual.** Nothing yet copies `instructions/` from here into a
consuming repo. The lock file *detects* drift; it does not fix it. Today you
copy the files and regenerate the hashes by hand.

**4. There is no build pipeline in the project** — only the review. The app's
real gates (`yarn lint`, `yarn build`, `yarn e2e-ci`) run on a developer machine,
not on an agent.

## Closing them, cheapest first

| Gap | Effort | Do it when |
|---|---|---|
| Rehearse the demo | 30 min | **Before showing anybody** |
| Say the miss out loud | 0 | It is a talking point, not a task |
| Add tenant-scoping to the prompt's priority list | 10 min | Before the next demo — likely fixes gap 2 |
| A build pipeline running lint + build + e2e-ci | 1 hour | Before anyone else commits |
| Automate the sync | half a day | Only once a second repo consumes this |

**Do not automate the sync yet.** One consuming repository does not need a
pipeline to copy four files. Build it when there are three repos and the copying
has actually hurt.

## What it costs

The reviewer is `gpt-4o` on a **Standard** deployment, 30K tokens per minute, in
a pay-as-you-go subscription. A review sends the diff plus the standards — around
15,000 tokens in, under 1,000 out. That is fractions of a cent per pull request.

Three levers if it ever matters:

1. **`gpt-4o-mini` for small diffs.** Same pipeline, change one variable.
2. **`queueOnSourceUpdateOnly`.** Today every push to a PR branch re-reviews.
   Turning it on reviews once per PR instead of once per push.
3. **The diff cap already exists** — 200KB, and the comment says when it truncated.

Azure DevOps itself is free at this size. When the demonstration is over:

```bash
az group delete --name rg-poc-ai-enablement --yes --no-wait
```

That deletes the model deployment, which is the only thing that bills. Leave the
repositories — they are the evidence.

## Where to read next

| Document | Answers |
|---|---|
| [`docs/how-it-works.md`](docs/how-it-works.md) | What happens between `git push` and a comment appearing |
| [`docs/file-by-file.md`](docs/file-by-file.md) | What every file contributes, repo level and org level |
| [`docs/working-without-ai.md`](docs/working-without-ai.md) | How to work here whether you use AI heavily, not at all, or somewhere between |

Half an hour, all three. They are written to be read once and then used as
reference, not to be admired.
