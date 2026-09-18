---
name: sdlc-intent
description: Capture an idea, ticket, bug report or production alert as a committed intent file — the problem, the outcome wanted, who and what it touches, the constraints, and what is still unknown. Use at the start of any piece of work, when someone says "I have an idea" or "we should build X", when users keep hitting something, when a ticket or incident needs turning into actionable work, or when asked to write a proposal, proto-spec or problem statement before any design or code. Covers the interview, the template, where the file lives, and when writing one is a waste of time.
---

# Capture the intent

First file in the chain (`ai-native-sdlc`). It exists so that an idea stops waiting on someone with the right job title to write it up, and so what reaches implementation still resembles what the originator actually meant.

## Is it worth writing?

**Yes** when a second person would need it written down to act, or when someone six months out will ask why this was built.

**No** for a typo, a rename, a one-line fix, or anything where the write-up costs more than the work. Judgment call, and erring toward "no" is cheaper than erring toward ceremony.

The originator can be anyone — the person with the idea, whoever triaged the ticket, or an agent that noticed a metric move (`sdlc-close-loop`). The route in doesn't change what gets written.

## Interview before template

The failure mode is reaching for the template immediately and filling it with the vague version of the idea.

Let them describe the problem however they describe it. What can't they do today? Who else hits this? What does better look like? What's explicitly not in scope? Then push where it's soft — "slow", "confusing" and "better UX" aren't outcomes, and a problem without a number in it usually hasn't been looked at yet.

Stop when you could hand it to someone who's never heard of it.

## The file

```markdown
# Intent: drafts survive a refresh

Raised by: T. Okafor (support). Status: draft.

## Problem
Long-form posts are lost when the editor tab reloads. 14 support
tickets this quarter; every one of them a user who had written
something substantial and had no way to get it back.

## Outcome wanted
An in-progress draft is recoverable after an unexpected reload,
without the author doing anything to save it.

## Touches
Editor, drafts API, whatever we pick for local persistence.

## Constraints
Cannot slow typing. Draft bodies must not land in analytics.

## Unknown
Do we restore silently, or prompt? Does this extend to the
mobile web editor, or is that a separate change?
```

Five things earn their place: **the problem** (with evidence), **the outcome**, **what it touches**, **the constraints**, **what's still unknown**. Anything else is optional and usually noise.

Leave the unknowns unresolved. Guessing at them here buries a decision that `sdlc-spec` should surface deliberately.

## Then

**Read it back.** The originator corrects whatever got mangled. This is the entire point of writing it down — skipping straight to commit produces a confident record of a misunderstanding.

**Commit it.** Author and timestamp attach themselves. A reviewer picks it up from there and decides whether it proceeds; that accept-or-close decision is the gate, and it belongs to a person.

## Where it lives

One product, one repo → a folder in that repo, so intent sits next to the code that came out of it. Monorepo → a directory. A separate repo only once intent genuinely spans many codebases; before that it's overhead.

Contributors without git don't need git — a version-control connector lets the agent commit markdown for them.

If a tracker already holds the record, decide which one is authoritative before you have two (`ai-native-sdlc`).

## Worth watching

How long from "someone had the thought" to a committed file. And the share of intents that get accepted rather than closed — a very high acceptance rate usually means the gate isn't real.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
