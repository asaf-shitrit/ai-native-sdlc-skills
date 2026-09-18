---
name: sdlc-plan
description: Write, interrogate and commit a plan before any code is generated — which files change, in what order, what could break, and what will prove it worked. Use before implementing anything non-trivial, when starting in plan mode, when asked "how would you approach this", when a change spans several files or carries blast radius, and when judging whether a session can safely run unattended. Covers the bar a plan has to clear, the questions that make one worth having, keeping plan and diff honest with each other, and when auto-accept is earned.
---

# Plan first

Third file in the chain (`ai-native-sdlc`). Left implicit, the approach lives in one person's head and the first reviewable thing is a finished diff — by which point changing direction means throwing work away. A plan moves that decision to when it's still a document edit.

## The bar

**Someone who never saw the conversation could implement the change from the plan alone.** Iterate until that's true. Most plans fail it on the first pass, usually by naming *what* without naming *where*.

Plan mode is what makes this real rather than aspirational: the codebase can be read but not edited, so the constraint is enforced by the tool and not by discipline.

## What a plan names

```markdown
# Plan: drafts survive a refresh

## Changing
web/src/editor/useDraftPersistence.ts   new
web/src/editor/Editor.tsx               mount the hook, restore on load
web/src/editor/__tests__/persistence.test.ts   new

## Order
1. Hook: write to IndexedDB on a debounce, read on mount.
2. Restore path in Editor, behind a prompt.
3. Clear persisted copy on successful publish.

## Risk
Debounce interacts with the existing autosave-to-server timer;
two writers to the same draft state. Step 1 has to make the
precedence explicit or drafts will flap between versions.

## Proof
persistence.test.ts covers restore-after-reload, publish-then-reload
(nothing restored), and the two-writer race. Manual: type, kill the
tab, reopen.
```

Four headings, and **Proof is not optional**. A plan without a checkable proof condition can't close its own loop (`sdlc-feedback-loop`), which puts a human back in the position of verifying everything by hand.

## Interrogate it

A plan accepted on first read wasn't worth writing. Ask:

- What does this break? Name something specific.
- Which step is riskiest, and why that one?
- What did you consider and reject?
- What don't you know yet?

The last question is the productive one. An honest "I don't know how X behaves under Y" is worth more than a confident plan built on a guess, and it's cheap to resolve now.

## Keeping it honest

Commit the approved plan. Review later checks the diff against it (`sdlc-pr-review`), which only works if the plan still describes reality.

So: **when implementation departs from the plan, the plan changes in the same commit.** Drift is normal and fine — silent drift is what makes the artifact worthless. A hook can enforce the pairing (`sdlc-hooks`).

## Earning auto-accept

Once a plan is approved, the agent can work through it without pausing per edit. That's appropriate when *all* of these hold:

- the plan is specific and the spec behind it is tight
- blast radius is small
- existing tests already cover the affected code
- the guardrails exist — a real `CLAUDE.md`, policy encoded where it matters, hooks on anything unrecoverable, a suite the agent can run itself

What it buys is the shift from watching edits scroll past to reviewing finished work, which is also what makes several sessions at once viable (`sdlc-parallel-work`).

Without the guardrails, auto-accept just means finding out later, in a bigger diff.

## Worth watching

The share of changes that merge without a second implementation pass. Later: how often the merged diff still matches the plan it claims to implement.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
