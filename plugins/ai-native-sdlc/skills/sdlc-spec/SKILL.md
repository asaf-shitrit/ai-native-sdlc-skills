---
name: sdlc-spec
description: Turn an accepted intent.md into a committed requirements-and-design spec.md in one working session, with policy applied while the spec is written and contradictions flagged rather than silently resolved. Use when an intent has been accepted and needs designing, when asked to write requirements, a design doc, a technical spec, or a PRD, when collapsing "requirements phase" and "design phase" into one pass, or when a stakeholder needs to review a design before engineering plans against it. Covers the prompt, the flagged-concerns discipline, and what the reviewer checks.
---

# Requirements and design as `spec.md`

Second artifact in the chain (see `ai-native-sdlc`). Requirements and design were separate phases run by separate teams for accountability reasons; the separation is slow and lossy. Here they collapse into one session, and the reviewer reviews rather than writes.

## Prerequisites

An accepted `intent.md` (`sdlc-intent`). Policies that must constrain the design — brand, security, compliance, UX — carry much more weight when they are written as skills (`sdlc-claude-md` covers when to write one), because then they apply while the spec is written instead of being discovered in a review weeks later.

## The pass

Start from the intent, with the org's policy skills loaded:

> Read the attached `intent.md` and produce a requirements and design spec for integrating it into our existing codebase. Apply the skills available to you so the plan conforms to our brand guidelines, security policies and UX standards. Document the spec fully as `spec.md`, ready to hand to the engineering team. Describe clearly any areas of concern, especially where you cannot satisfy contradicting policies.

Run it by hand the first few times. Then codify it as a slash command. The end state is a non-interactive job fired by the intent's merge that opens `spec.md` as a PR — at which point the product owner's first involvement is the review.

## Flagged concerns are the deliverable

The single most valuable output is not the design, it is the list of things that cannot be cleanly satisfied. These are the points an analyst would have escalated.

**Flag, do not resolve, when:**
- Two policies contradict each other for this change
- The intent's constraints cannot all hold at once
- An open question from `intent.md` still has no answer
- The design needs a decision that belongs to a named policy owner

Say plainly what the conflict is and who owns each side. A spec that quietly picks a winner has hidden a decision that a human was accountable for.

**Work the flagged concerns first.** Each one goes to its policy owner and gets resolved *before* engineering sees the spec.

## What the reviewer checks

Not prose quality. Two things:

1. Does the spec solve the problem stated in `intent.md`?
2. Are the intent's open questions answered, or explicitly carried forward?

Then: commit `spec.md` alongside `intent.md`. The pair records what was asked for and what was decided.

## The gate

A human decides whether spec and intent progress to build, consulting a technical lead for anything the organization classes as higher risk. Accepting the spec is what starts planning (`sdlc-plan`).

## Front-end work

The clearest case for compressing the phases: mock the design from `intent.md`, iterate on the mock, then export it to the implementation session. The approved mock becomes the proof condition in `plan.md` and the target for the visual loop in `sdlc-feedback-loop`.

## Governance

The spec, the prompt that produced it, and the versions of the skills in force are all in version control. Policy is read and applied at authoring time. The reviewer signs off; flagged concerns route to named owners.

## Measuring it

- **Leading** — elapsed time between the `intent.md` commit and the `spec.md` commit for the same change (two git timestamps), against the old requirements-plus-design cycle.
- **Lagging** — requirements rework after build starts: count `spec.md` commits dated after the first `plan.md` commit for the same change. `git log` gives this directly.
