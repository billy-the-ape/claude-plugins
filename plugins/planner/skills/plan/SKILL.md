---
name: plan
description: Work out how to implement a change before writing any of it — scope, the files involved, the approach, the risks, and an ordered sequence of steps. Use this whenever the user asks for a plan, asks how to approach or structure a change, says they want to think an implementation through before coding, or describes a non-trivial feature, migration or refactor — even when they never use the word "plan".
---

# Plan

> **This body is a stub.** Replace it with your own planning prompt. The frontmatter
> above is what decides *whether* the skill runs; everything below decides *what it
> does*. Keep the description in sync when you change the scope.

## Read before you propose

A plan written from the request alone tends to describe a codebase that does not
exist. Ground it first:

- Find the code that already does the nearest thing, and follow its conventions.
- Name the actual files and functions the change touches.
- Check whether an existing abstraction covers this before adding a new one.

## Shape of the plan

Produce something executable, not a description of the goal restated:

1. **Scope** — what changes, and explicitly what does not.
2. **Approach** — the design, and the alternative you rejected and why.
3. **Steps** — ordered, each independently verifiable.
4. **Risks** — what could break, and how it would show up.
5. **Validation** — the commands or checks that prove each step landed.

## Where plans go wrong

- Steps that cannot be verified until the very end. Sequence so each one is checkable.
- Unstated assumptions about behaviour nobody confirmed. Verify against source, or
  mark it clearly as an assumption.
- Scope that quietly grows past what was asked. Note adjacent problems; don't absorb them.

Stop at the plan. Do not start implementing unless the user asks.
