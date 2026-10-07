---
name: review
description: Review a diff, branch, or pull request for correctness bugs, regressions, and risky changes, post findings as inline comments, and decide whether the change is approvable. Use this whenever the user asks for a code review, asks you to look over a change before it merges, mentions reviewing a diff, a branch or a PR, or asks whether a change is safe to ship — even when they never use the word "review".
allowed-tools: Read Grep Glob Bash(git diff *) Bash(git log *) Bash(git status *) Bash(git rev-parse *) Write(./.claude-review-verdict.json)
---

# Review

This skill reviews and judges. It never changes code.

That separation is the point: when the same agent writes a fix, decides the fix is
good, and clears the merge, there is no independent check anywhere in the loop. So
the tool grant above has no `Edit`, no `git commit`, no `git push`, and no merge —
not as a matter of policy you could talk yourself out of, but because the tools are
absent. The one writable path is the verdict file, which the workflow turns into a
real GitHub review. Nits you would otherwise fix in place go to the backlog instead;
`/reviewer:sweep` clears them later in a PR a human can read.

## Establish the target first

The wrong base makes every finding suspect, so settle this before reading code:

- An explicit target (`owner/repo/pull/N`, a branch, a path, a ref range) wins.
- Otherwise review the current branch against the PR's base, which is not always the
  default branch. `git rev-parse` the base before diffing against it.
- If you cannot determine the base, say so and stop. Do not guess `main`.

## Severity, and what it costs to get wrong

Rank every finding, because the author acts on the ranking:

- **Important** — would break behaviour, lose data, or leak something in production.
  A finding earns this only when you can name the input or state that produces the
  wrong result. "This looks fragile" is not an Important finding.
- **Nit** — correct but worth improving. Never blocks.
- **Pre-existing** — real, but this change did not introduce it. Label it so, or the
  author wastes a round trip discovering it is not theirs.

Inflating a nit to Important costs the author a cycle and teaches them to distrust
the next review. Burying a real bug among nits costs more. Spend the care here.

## What not to report

Noise is the failure mode that makes reviews get ignored:

- Anything CI already enforces — formatting, lint, type errors.
- Style preferences with no behavioural consequence.
- Anything you could not verify in the code you actually read. If a claim rests on
  what a function's name suggests rather than what its body does, drop it.

## The verdict

After posting inline findings, write `./.claude-review-verdict.json`:

```json
{
  "event": "APPROVE",
  "summary": "One or two sentences the PR author reads first.",
  "important": 0,
  "nits": 3
}
```

`event` follows from the findings, with no discretion:

- **`REQUEST_CHANGES`** — one or more Important findings.
- **`APPROVE`** — no Important findings. Nits alone never block; record them in the
  backlog and approve.
- **`COMMENT`** — you could not complete the review (base unresolvable, diff too
  large to read honestly). Say why in `summary`. Never approve a change you did not
  finish reading.

Write the file exactly once, as the last thing you do. The workflow submits the
review from it, so a missing file means no review was posted at all — if you are
stopping early, still write the file with `COMMENT`.

## Nit backlog

For each nit, include a line in `summary` of the form `nit: <path>:<line> — <what>`.
The sweep job collects these across PRs and fixes them in one pass, which keeps this
job read-only and keeps mechanical cleanups out of the review history.
