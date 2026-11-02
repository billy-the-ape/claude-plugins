---
name: review
description: Review a diff, branch, or pull request for correctness bugs, regressions, and risky changes, post findings as inline comments, and decide whether the change is approvable. Use this whenever the user asks for a code review, asks you to look over a change before it merges, mentions reviewing a diff, a branch or a PR, or asks whether a change is safe to ship — even when they never use the word "review".
allowed-tools: Skill Agent Task Read Grep Glob Bash(git diff *) Bash(git log *) Bash(git status *) Bash(git rev-parse *) Write(./.claude-review-verdict.json)
---

# Review

This skill adds only what a caller needs around a review. The review itself is
delegated, and the restrictions are enforced by the caller's tool grant rather
than restated here: there is no `Edit`, no `git commit`, no `git push`, no merge tool
and no build or test command, so none of those is available to talk yourself into.

## Delegate the review itself

Run `/code-review:code-review <target>` and let it do the review. Its
prompt is maintained and evaluated upstream; a second copy of that reasoning kept
here would drift from it and be the worse of the two within a month.

Add `--comment` only when your own arguments include it. A caller that submits
the review from your verdict, as the shared workflow does, leaves it off: that
skill then stops at its summary instead of posting comments of its own beside
the review the caller submits. When you turn its findings into inline comments
for the verdict, follow its commenting rules anyway: one comment per unique
issue, brief, and a suggestion block only when committing it fixes the issue
entirely.

Everything below is what that skill cannot know about this setup.

## Run its subagents

That skill is a pipeline of subagents: parallel reviewers, then a validator per
finding. Launch them as it describes rather than doing its steps yourself in one
pass. A single pass loses the independent second look and the validation step,
which is why a clean review could be followed by one that finds real issues.

Launch every subagent in the foreground: pass `run_in_background: false` on
each Agent call, and send a step's parallel agents in one message so they still
run side by side. This review runs non-interactively in CI, so the run ends the
moment you end your turn. Nothing wakes you when a background agent finishes,
its findings are lost, and no verdict is written. Never end your turn while a
subagent is still running, and never end it before you have given the verdict.

Tell each subagent to run one plain command per Bash call. The tool grant
approves single commands, not pipes, redirects, `;`, `&&` or shell variables,
so a compound command is denied and wastes a turn. Run git from the working
directory, never with `git -C <path>`: the grant matches commands that start
`git diff`, `git log` and so on, so a `-C` form is denied.

Run every agent that skill asks to run on Opus on Sonnet instead
(`model: "sonnet"`): the bug-finding reviewers and the validators alike. This
is a cost choice for this setup: Opus agents were most of a review's usage.
Agents that skill already puts on Haiku or Sonnet keep their model.

## Re-reviews are expected

Its first step stops when Claude has already commented on the pull request. In
this setup that condition does not apply: the workflow re-reviews on every push,
and the earlier review from `github-actions[bot]` covers an older commit. Its
other stop conditions still apply; if one stops the review, give the verdict
`COMMENT` and name the condition in `summary`.

## CI has already run — read it, do not reproduce it

The caller tells you CI's conclusion and where its logs are. Treat that as the
source of truth for whether the code builds and the tests pass. You have no build
or test command, which is deliberate: re-running a suite CI just ran spends
minutes and tokens to learn something already on the page.

A red CI does not stop the review. Read the failure and say whether it looks
caused by this change or independent of it — that is the most useful thing you can
tell an author staring at a red check, and a failure unrelated to the diff is not
a reason to withhold approval.

## Find nits in a pass of their own

That skill drops nits on purpose: its reviewers and validators are told not to
flag style, quality concerns or pedantic nitpicks. A nit you note while
orchestrating would skip every check the pipeline applies, so once its review
has finished, give nits a pass of their own:

1. Launch one nit-finding agent on Sonnet. Give it the target, the pull
   request's title and description, and the findings the review kept, so it
   does not report those again as nits. It looks only at lines the change adds
   or modifies, for the kinds of nit `/reviewer:sweep` can clear: naming, dead
   code, duplicated literals, a missing early return, a comment that no longer
   matches the code beside it, something a linter would catch if the rule were
   enabled. Not pre-existing code, not anything CI or a configured linter
   already enforces, and not matters of taste. For each nit it returns the
   path, the line in the new version of the file, a one-line note, and — only
   when one change at that location fixes it entirely — the exact replacement
   lines.
2. Launch one validating agent on Sonnet with that whole list, plus any nits
   you noticed yourself. It reads each cited line and drops any nit that is
   wrong, pre-existing, already covered by a kept finding, or not worth a
   reviewer's time.
3. Keep at most ten, the ones most worth fixing.

Both agents follow the rules above: foreground, one plain command per Bash
call, no `git -C`. Report only the nits the validator kept.

## Record nits, do not fix them

Put each kept nit in the verdict's `nits` array instead of acting on it; a
separate sweep batches them into one pull request later. If the caller asks for
mechanical nits as suggestion comments instead, route those the way it asks.
Report the same nit the same way each time you see it, because a re-review
replaces a pull request's recorded nits wholesale — a nit rephrased on a second
pass reads as new, and the backlog accumulates duplicates instead of converging.

## Write the verdict last

Finish with the verdict. A caller that runs this skill with `--json-schema`, as
the shared workflow does, receives it as structured output: return it that way
and write no file. Otherwise write it to `./.claude-review-verdict.json`. The
fields are the same either way:

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

The caller submits a real GitHub review from the verdict, so a missing verdict
means no review was posted at all. If you are stopping early, still give it with
`COMMENT`.
