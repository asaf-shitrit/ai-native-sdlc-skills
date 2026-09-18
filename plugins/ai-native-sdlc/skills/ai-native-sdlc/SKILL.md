---
name: ai-native-sdlc
description: Router and shared model for the AI-native software development lifecycle — the committed-artifact chain (intent.md → spec.md → plan.md → diff+tests → PR findings → incident record) that replaces ticket-and-handoff process when an agent writes most of the code. Use when deciding how to run a piece of work end to end, when asked "how should we structure this change", when setting up or improving a project's agentic workflow, when the build is fast but planning/review/deploy are the bottleneck, or when picking which sdlc-* skill applies. Also use for adoption order and for the leading/lagging metrics that say whether the change worked.
---

# The AI-native SDLC

## The one idea

Code is no longer the bottleneck. The steps to the **left and right** of build — plan, review, test, deploy — still run at human speed, so they become the constraint. Traditional controls (read every line, route exceptions through a weekly meeting) stop matching reality once an agent writes most of the diff.

The fix is not "less process." It is **the same control objectives with new enforcement**: every stage ends by committing an artifact the next stage reads, and the chain of commits *is* the audit trail — who asked for what, what the agent produced, who approved it.

## The artifact chain

```
idea / ticket / alert
   ↓  intent.md      what is wanted, why, under what constraints   [sdlc-intent]
   ↓  spec.md        requirements + design, concerns flagged       [sdlc-spec]
   ↓  plan.md        files, order, risks, proof                    [sdlc-plan]
   ↓  diff + tests   implementation that verified itself           [sdlc-feedback-loop]
   ↓  PR + findings  agentic review passes, human judges risk      [sdlc-pr-review]
   ↓  production     control bands watch it                        [sdlc-close-loop]
   ↺  breach → new intent.md
```

Each accepted artifact is the trigger for the next stage. Start by prompting each step by hand; the end state is a loop where the merge fires the next gate and human attention concentrates **at the gates**, reviewing what the agent flagged rather than starting each stage cold.

`.md` artifacts up front because a product owner and an agent can both read and act on the same file. From build onward the artifact is code and its records.

## Which skill

| Situation | Skill |
|---|---|
| Someone has an idea, a ticket landed, an alert fired | `sdlc-intent` |
| Turning accepted intent into a design the team can plan against | `sdlc-spec` |
| About to implement anything non-trivial | `sdlc-plan` |
| Agent keeps making the same mistake; encoding conventions or policy | `sdlc-claude-md` |
| Agent can't tell whether its own work is correct | `sdlc-feedback-loop` |
| Changing CLAUDE.md, skills, hooks, or the model | `sdlc-agent-evals` |
| A rule must hold without exception, or a human must approve | `sdlc-hooks` |
| More work in flight than one session can carry | `sdlc-parallel-work` |
| PRs queueing, review quality varying | `sdlc-pr-review` |
| Production signal should start work without a person in the path | `sdlc-close-loop` |

## Adoption order

Plays are modular; adopt by dependency, not by stage number. Nothing points into these, so start anywhere here:

- **`sdlc-claude-md`** — cheapest, compounds into everything else
- **`sdlc-feedback-loop`** — the single highest-leverage play; without it a human checks all agent output
- **`sdlc-plan`** — free, just a habit
- **`sdlc-intent`** — needs an agreed template and a home
- **`sdlc-hooks`** — no prerequisites

Then, in rough order: `sdlc-parallel-work` (needs CLAUDE.md + feedback loop) → `sdlc-pr-review` (needs CLAUDE.md, helped by skills) → `sdlc-spec` (needs intent + policy skills) → `sdlc-agent-evals` (needs CLAUDE.md + feedback loop) → `sdlc-close-loop` (needs intent format, review gate, hooks, a rehearsed rollback).

## Where the human stays

Humans remain accountable for every decision requiring judgment. What moves is **which artifact they read**: not the diff line by line, but the intent, the flagged concerns, the plan, and the ranked findings. Separation of duties survives because the agent that wrote the code has no route to approve it.

## Source of truth, when legacy tools already hold the record

Jira, a requirements tool, Figma, a change board — these are hard to displace because auditors already accept them. For **every artifact**, name exactly one system as the source of truth:

- **Repo as source of truth** — markdown is authoritative, legacy system links to commits. Cleanest for engineering-led teams: one tool, one timestamp authority.
- **Legacy as source of truth** — the tool holds the record, markdown files are working copies; Claude reads the record at session start and writes the outcome back over MCP in the same session.
- **Linkage as the minimum bar** — every artifact notes the record ID, every record holds the commit SHA. Two sources of truth, honestly labeled. Fine as a starting point.

## Measuring it

Every play has both. Prefer metrics the toolchain already emits (git timestamps, PR metadata, CI results, incident tracker) over anything self-reported.

- **Leading** — elapsed time between two artifacts in the chain. Intent→spec, spec→plan, plan→merge, breach→triage queue.
- **Lagging** — rework and escape rate. Spec commits dated after the first plan commit; merged diffs that no longer match plan.md; defects caught before merge vs. escaping to production; repeat incidents of the same class.

A leading indicator that improves while the lagging one worsens means the process is going faster at producing rework. Read them as a pair.

---

*Distilled from Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) by Louis Claxton (August 2026), which is the canonical source. This is an unofficial repackaging into skill form; not affiliated with or endorsed by Anthropic.*
