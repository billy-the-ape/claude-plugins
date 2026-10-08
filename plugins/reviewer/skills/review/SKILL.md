---
name: review
description: Review a diff, branch, or pull request for correctness bugs, regressions, and risky changes, post findings as inline comments, and decide whether the change is approvable. Use this whenever the user asks for a code review, asks you to look over a change before it merges, mentions reviewing a diff, a branch or a PR, or asks whether a change is safe to ship — even when they never use the word "review".
allowed-tools: Skill Read Grep Glob Bash(git diff *) Bash(git log *) Bash(git status *) Bash(git rev-parse *) Write(./.claude-review-verdict.json)
---

# Review

This skill adds only what a caller needs around a review. The review itself is
delegated, and the restrictions are enforced by the tool grant above rather than
restated here: there is no `Edit`, no `git commit`, no `git push`, no merge tool
and no build or test command, so none of those is available to talk yourself into.

## Delegate the review itself

Run `/code-review:code-review --comment <target>` and let it do the review. Its
prompt is maintained and evaluated upstream; a second copy of that reasoning kept
here would drift from it and be the worse of the two within a month.

Everything below is what that skill cannot know about this setup.

## CI has already run — read it, do not reproduce it

The caller tells you CI's conclusion and where its logs are. Treat that as the
source of truth for whether the code builds and the tests pass. You have no build
or test command, which is deliberate: re-running a suite CI just ran spends
minutes and tokens to learn something already on the page.

A red CI does not stop the review. Read the failure and say whether it looks
caused by this change or independent of it — that is the most useful thing you can
tell an author staring at a red check, and a failure unrelated to the diff is not
a reason to withhold approval.

## Record nits, do not fix them

Put every nit in the verdict's `nits` array instead of acting on it; a separate
sweep batches them into one pull request later. Report the same nit the same way
each time you see it, because a re-review replaces a pull request's recorded nits
wholesale — a nit rephrased on a second pass reads as new, and the backlog
accumulates duplicates instead of converging.

## Write the verdict last

Finish by writing `./.claude-review-verdict.json`:

```json
{
  "event": "APPROVE",
  "summary": "One or two sentences the PR author reads first.",
  "important": 0,
  "nits": [
    { "path": "src/video/worker.ts", "line": 142, "note": "claimedAtMs has no reader" }
  ]
}
```

- **`REQUEST_CHANGES`** — at least one finding you would block a merge on.
- **`APPROVE`** — no blocking findings. Nits alone never block.
- **`COMMENT`** — you could not complete the review. Say why in `summary`. Never
  approve a change you did not finish reading.

The caller submits a real GitHub review from this file, so a missing file means no
review was posted at all. If you are stopping early, still write it with `COMMENT`.
