---
name: review
description: Review a diff, branch, or pull request for correctness bugs, regressions, and risky changes, then report findings ranked by severity. Use this whenever the user asks for a code review, asks you to look over a change before it merges, mentions reviewing a diff, a branch or a PR, or asks whether a change is safe to ship — even when they never use the word "review".
---

# Review

> **This body is a stub.** Replace it with your own review prompt. The frontmatter
> above is what decides *whether* the skill runs; everything below decides *what it
> does*. Keep the description in sync when you change the scope.

## Work out what you are reviewing first

Establish the target before reading any code, because the wrong base makes every
finding suspect:

- An explicit target (`owner/repo/pull/N`, a branch, a path, a ref range) wins.
- Otherwise review the current branch against its base, plus uncommitted changes.
- If the base branch is not obvious, say so rather than assuming `main`.

## What to report

Lead with correctness. A finding earns its place when you can name the input or
state that produces the wrong behaviour — not when the code merely looks unusual.

Rank findings so the author knows what to act on first:

- **Important** — would break behaviour, lose data, or leak something in production.
- **Nit** — worth fixing, never blocking.
- **Pre-existing** — real, but this change did not introduce it. Say so explicitly.

## What not to report

Noise costs the author more than a missed nit costs you:

- Anything CI already enforces — formatting, lint, type errors.
- Style preferences with no behavioural consequence.
- Speculation you could not verify in the code you read.

## Reporting

For each finding give the location, one sentence on the defect, and the concrete
failure case. If you found nothing important, say that plainly and first.
