---
name: sdlc-intent
description: Capture an idea, ticket, bug report, or production alert as a committed intent.md proto-spec — what is wanted, why, who and what it affects, under which constraints, and what is still open. Use at the very start of any piece of work, when someone says "I have an idea", "we should build X", "users keep complaining about Y", when a ticket or incident needs turning into actionable work, or when asked to write up a proposal, proto-spec, or problem statement before any design or code. Covers the template, the brainstorm-first interview, where intent lives in the repo, and when NOT to bother.
---

# Capture intent as `intent.md`

The first artifact in the chain (see `ai-native-sdlc`). It exists so an idea stops waiting for someone to write it up, and so what reaches engineering is the originator's own meaning rather than something four handoffs removed from it.

## When this applies

Any of these routes produce an intent:

- A person has an idea and describes it in their own words
- A ticket is filed
- An alert or breached control band surfaces a problem (see `sdlc-close-loop`)
- A security or codebase finding is too big to fix in one PR

**Skip it** for a typo, a one-line fix, or anything where writing the intent costs more than doing the work. The test: would a second person need this written down to act on it, or to judge later whether it was the right thing to build?

## How to run it

1. **Let them describe the problem in their own words.** No formal language. What they cannot do today, who is affected, what better looks like, what is out of scope. Do not reach for the template yet.
2. **Brainstorm until the idea is concrete.** Ask what an analyst would ask: scope, users, constraints, success criteria, what happens today instead. Push on vagueness — "faster" and "better UX" are not outcomes. Stop when you could hand it to someone who has never heard of it.
3. **Write it as `intent.md`** using the template below (or the org's, if one is encoded as a skill).
4. **Read it back and let the originator correct anything you misunderstood.** This is the whole point of the artifact; do not skip to committing.
5. **Commit it** to the intent home. Author and timestamp join the record.

## Template

```markdown
# Intent: claims status self-service

Author: J. Ortiz (claims operations). Status: draft.

## Problem
Customers phone the contact center to ask where their claim is.
Handlers spend roughly a third of call time on status-only queries.

## Proposed outcome
Customers see claim status, next step and expected date in the portal.

## Affected users and systems
Claims handlers, portal team, claims-core API.

## Constraints
No new PII in the portal session. Existing authentication only.

## Open questions
Do third-party loss adjusters need access too?
```

Sections that earn their place: **Problem** (with a number in it wherever possible), **Proposed outcome**, **Affected users and systems**, **Constraints**, **Open questions**. Anything else is optional.

Carry open questions forward rather than guessing at them — `sdlc-spec` either answers them or escalates them.

## Where it lives

- **Single product** → an `intent/` folder in the product repo. Simplest, and it keeps the artifact chain next to the code derived from it.
- **Monorepo** → a directory.
- **Intent spanning many repos** → a dedicated intent repo, but only then; it is overhead otherwise.

Non-engineers do not need git. A version-control connector (e.g. the GitHub connector on claude.ai) lets Claude commit the markdown on their behalf.

If a ticketing tool already holds the record, decide which is authoritative — see the source-of-truth section in `ai-native-sdlc`.

## Governance

The committed file *is* the evidence: author, timestamp, full revision history. The accept/reject decision that promotes an intent to design is recorded as the merge or the closing review. A product owner makes that call, not the agent.

## Measuring it

- **Leading** — time from first conversation to a committed `intent.md`, read off git history. Expect weeks of elicitation to collapse to hours.
- **Lagging** — survival rate: the share of intents accepted into design rather than closed. Plus the number of edits to `intent.md` made *after* the first `spec.md` commit, which measures how much was missed the first time.

---

*Distilled from Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) by Louis Claxton (August 2026), which is the canonical source. This is an unofficial repackaging into skill form; not affiliated with or endorsed by Anthropic.*
