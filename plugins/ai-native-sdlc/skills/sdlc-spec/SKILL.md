---
name: sdlc-spec
description: Turn accepted intent into a committed spec in one working session, applying policy while the design is written and surfacing contradictions instead of quietly resolving them. Use when an intent has been accepted and needs designing, when asked to write requirements, a design doc, a technical spec or a PRD, when collapsing separate requirements and design phases into a single pass, or when someone needs to review a design before implementation is planned. Covers the pass itself, the flagged-conflict discipline, and what the reviewer is actually checking.
---

# From intent to spec

Second file in the chain (`ai-native-sdlc`). Requirements and design were separate phases owned by separate people for accountability reasons. The accountability is worth keeping; the two-phase handoff is not — it's slow, and meaning leaks at the boundary.

Here it's one pass, and the reviewer reviews instead of writing.

## Before you start

An accepted intent file (`sdlc-intent`). Whatever policy must constrain the design — security rules, API conventions, accessibility, brand — carries far more weight when it's written down as a skill (`sdlc-claude-md`), because then it shapes the design as it's written rather than being discovered in review three weeks later.

## The pass

Load the intent and the policy skills, then ask for a design that:

- solves the problem the intent states, in the existing codebase, not in the abstract
- conforms to the policies in force, naming which one drove which decision
- **calls out every place two requirements or two policies can't both hold**

Run it by hand until you trust the shape, then make it a slash command. Eventually the intent's acceptance triggers it and opens the spec as a PR — at which point the first human involvement is the review.

## Conflicts are the deliverable

The design is the boring part. The valuable output is the list of things that can't be cleanly satisfied, because those are exactly what a careful analyst would have escalated.

Surface rather than solve when:

- two policies pull in opposite directions for this change
- the intent's constraints can't all hold simultaneously
- an unknown from the intent is still unknown
- the decision belongs to someone who isn't in the room

State the conflict plainly and name who owns each side. **A spec that silently picks a winner has hidden a decision a person was accountable for** — and it will resurface during implementation, when reversing costs more.

Resolve these first, with their owners, before anyone plans against the spec.

## What the review is for

Not prose quality. Two questions:

1. Does this solve the problem the intent described?
2. Is every unknown either answered or deliberately carried forward?

Then commit the spec next to the intent. The pair is the record: what was asked for, and what was decided.

Progressing to implementation is a human decision, with a technical lead pulled in for anything the team treats as higher-risk.

## Design-heavy work

For anything visual, iterate on a mock from the intent before writing the spec. The approved mock then becomes the proof condition in the plan (`sdlc-plan`) and the comparison target for the visual loop (`sdlc-feedback-loop`) — which is what makes UI work verifiable at all.

## Worth watching

Elapsed time between the intent commit and the spec commit. Then, later: spec edits landing *after* implementation began, which is the direct measure of what this pass failed to pin down.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
