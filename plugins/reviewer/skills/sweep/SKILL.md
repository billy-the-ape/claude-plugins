---
name: sweep
description: Clear accumulated review nits across a codebase in one pass — mechanical cleanups that were noted during PR reviews but deliberately not fixed there — and open a single pull request with the result. Use this when the user asks to clean up nits, clear the nit backlog, do a tidy-up or janitorial pass, or fix the small things that reviews keep flagging.
allowed-tools: Read Grep Glob Edit Write Bash(git status *) Bash(git diff *) Bash(git log *) Bash(git checkout *) Bash(git add *) Bash(git commit *) Bash(git push *)
---

# Sweep

This is where the write access lives, deliberately kept out of `/reviewer:review`.
Everything here lands on a branch of its own, for one pull request — never a push to
a protected branch, and never a push to someone else's PR branch. A human reads the
result before it merges, which is what makes it safe for this job to edit code at
all.

## Pick up only what is safe to batch

Take a nit only when the fix is mechanical and its correctness is obvious from the
diff alone:

- Naming, dead code, duplicated literals, a missing early return.
- A comment that no longer matches the code beside it.
- Something a linter would catch if the rule were enabled.

Leave anything that needs a judgement call about behaviour. A sweep PR gets reviewed
quickly precisely because every change in it is boring; one real behavioural change
hidden in forty cosmetic ones is how a sweep PR becomes unreviewable and sits for a
week.

## Order of work

1. Branch from the current default branch. Never work on an existing PR branch.
2. Group the changes by concern, one commit each. A reviewer reads this PR by
   commit, so "rename the misleading variable in the worker" beats one commit of
   two hundred unrelated lines.
3. Run the repository's own fast checks before pushing — whatever a contributor runs
   locally. A sweep PR that turns CI red has cost more than it saved.
4. Push the branch. The branch is the deliverable: a caller may open the pull
   request itself, so do not assume you are the one opening it — check whether
   you were told to, and if you were not, report what you fixed and what you
   left so the caller can write it up.

## What to leave behind

Say explicitly which nits you skipped and why, so the next sweep does not re-examine
them from scratch and so a human can see whether the backlog is being worked or just
churned. A nit you decline twice for the same reason should be reported as something
to stop flagging, not carried forever.
