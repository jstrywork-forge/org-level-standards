# Working here, whatever your setup

Three developers, three postures, one repository. None of them should be slowed
down by the other two.

| Posture | What this repo asks of you |
|---|---|
| **No AI at all** | Nothing. Read two markdown files once |
| **Occasional AI** | Nothing. It reads them for you |
| **A curated setup of your own** | Nothing. Point it at the same files |

The design goal is stated once, plainly: **the instruction layer is documentation
that a machine happens to be able to read.** If it only works for people with a
particular tool configured, it has failed.

---

## If you do not use AI

`.github/copilot-instructions.md` and `.github/instructions/org/*.md` are
markdown. Read them once when you join, skim the org ones when the version bumps.
That is the whole obligation.

**Nothing about your workflow changes:**

- The build gates your pull request. It always did.
- The review comment is advisory and never blocks a merge.
- If a comment is wrong, reply saying so and merge. It is a colleague's opinion,
  and a fallible one.
- Nothing installs. Nothing phones anywhere from your machine.

The only thing you will notice is a comment on your pull request from the build
service, which you may ignore.

**You still benefit**, and it is worth being honest about how: the standards you
would otherwise have to remember are now applied consistently to everyone's pull
requests, including the ones written by people who joined last week.

## If you use Copilot as it comes

Nothing to configure. Open the folder; the instructions attach themselves.

Two things worth knowing:

**It will suggest 2020 Angular** unless told otherwise — `NgModule`,
`*ngIf`, `@Input()`, constructor injection. The org file exists mostly to correct
that. If you see those in a suggestion, the instruction did not attach; check you
opened the repository root and not a subfolder.

**Repo beats org.** Where the two disagree, follow the repo file. If you find a
conflict that is not written down, write it down.

## If you have a curated setup of your own

A different assistant, custom prompts, your own harness — fine. The files are
plain markdown with YAML front matter and no tool-specific syntax beyond
`applyTo:`, which you can ignore or map.

**Point it at:**

```
.github/copilot-instructions.md      repo conventions
.github/instructions/org/*.md        org standards
.github/instructions/org.lock.json   which version, and hashes
```

**One rule that matters:** do not let your setup *edit* anything under
`instructions/org/`. The next sync overwrites it, and the drift check will report
your machine as the source of a mismatch. Propose changes in the standards repo.

If your setup is better than what the pipeline does, say so — the pipeline's
reviewer is one `curl` and deliberately replaceable.

---

## What is genuinely required of everyone

Short list, and it is short on purpose.

1. **Run `yarn lint` before you push.** `yarn start` does it for you.
2. **Do not hand-edit `src/app/shared/api/`.** It is generated. Change
   `contract/openapi.json` and regenerate.
3. **Do not import `@angular/material` outside `ui-kit`.** Lint stops you; the
   reason is that swapping component libraries stays a one-project change.
4. **Do not edit anything under `.github/instructions/org/`.**

Everything else is guidance. Those four are the repository's actual rules, and
three of the four are enforced by tooling rather than by trust.

## The failure this is designed to avoid

An instruction layer that makes people slower is worse than none. Specifically:

- **A review that blocks.** A gate that is sometimes wrong gets routed around
  within a month, and you lose the whole thing defending one bad comment. This
  one cannot block; the policy is set to optional.
- **A review that comments on everything.** The prompt contract forbids repeating
  anything the build already enforces, and forbids style opinions where the repo
  is internally consistent.
- **Instructions nobody can find.** Two files, both under `.github/`, both
  linked from the README.
- **Rules with no reason.** Every standard says what it prevents. A rule whose
  reason is not recorded is one nobody can argue with — and therefore one nobody
  can safely remove.

If any of those start being true, that is a bug in this setup, not in you. Raise
it in the standards repo.
