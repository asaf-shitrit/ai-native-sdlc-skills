---
name: ai-native-sdlc
description: Router and shared model for running software work as a chain of committed artifacts — intent, spec, plan, verified diff, reviewed PR, monitored production — so that planning, review and deploy keep pace when an agent writes most of the code. Use when deciding how to run a piece of work end to end, when asked "how should we structure this change", when setting up or improving a project's agentic workflow, when the build is fast but everything around it is slow, or when picking which sdlc-* skill applies. Also covers what to adopt first and which numbers say it worked.
---

# Running the lifecycle as an artifact chain

## Pick the skill

| You are here | Go to |
|---|---|
| An idea, ticket or alert needs turning into actionable work | `sdlc-intent` |
| Accepted intent needs a design the team can plan against | `sdlc-spec` |
| About to implement something non-trivial | `sdlc-plan` |
| The agent repeats mistakes, or a convention is applied inconsistently | `sdlc-claude-md` |
| The agent can't tell whether its own work is correct | `sdlc-feedback-loop` |
| Changing the config that steers the agent, or the model | `sdlc-agent-evals` |
| A rule must hold every time, or a named human must approve | `sdlc-hooks` |
| More work in flight than one session can carry | `sdlc-parallel-work` |
| PRs queueing, or review depth varying by reviewer | `sdlc-pr-review` |
| Production signal should start work with nobody in the path | `sdlc-close-loop` |

## Why the chain

Generating code got cheap. Everything bracketing it — agreeing what to build, judging whether the result is right, getting it shipped — did not. So those become the constraint, and the old controls stop fitting: reading every line was reasonable when a person typed every line.

Replacing the controls with nothing is not the answer. Replacing the *enforcement* is. Each step leaves behind a file the next step reads:

```
   intent      the problem, in the originator's words       sdlc-intent
     ↓
   spec        what to build, contradictions surfaced       sdlc-spec
     ↓
   plan        files, sequence, risk, proof                 sdlc-plan
     ↓
   diff        code that already checked itself             sdlc-feedback-loop
     ↓
   PR          uniform review passes, ranked findings       sdlc-pr-review
     ↓
   prod        watched; a breach writes the next intent     sdlc-close-loop
     ↑___________________________________________________________|
```

Three properties make this work:

- **Each artifact is a handoff.** The next step reads a file, not a memory of a meeting.
- **The commit log is the record.** Who asked, what was produced, who approved — already captured, no separate audit exercise.
- **Judgment stays human, but moves.** Nobody reads every line. They read the intent, the flagged contradictions, the plan, the ranked findings.

Markdown up front, because a file is the one format a person and an agent can both act on. From the diff onward the artifact is the code and its records.

## What to adopt first

Not in stage order — by dependency. Four skills need nothing:

- **`sdlc-claude-md`** — cheapest, compounds into every other one
- **`sdlc-feedback-loop`** — the one with the largest effect. Skip it and a human verifies everything, which is the bottleneck you were trying to remove
- **`sdlc-plan`** — a habit, not infrastructure
- **`sdlc-hooks`** — for the single rule that must never break

After those: `sdlc-parallel-work` wants the first two in place. `sdlc-pr-review` wants a real `CLAUDE.md`. `sdlc-spec` wants intent plus whatever policy you've encoded. `sdlc-agent-evals` wants something to measure. `sdlc-close-loop` wants nearly all of it, plus a rollback you've actually rehearsed.

## Legacy tools already hold the record

Ticket trackers and requirements tools are hard to displace — auditors accept them and other teams depend on them. Pick one authority *per artifact*, then be consistent:

- **Repo authoritative** — markdown rules, the tracker links to commits. Cleanest when engineering owns the process end to end.
- **Tracker authoritative** — the tool holds the record; markdown files are working copies; the agent reads at session start and writes back over MCP in the same session.
- **Cross-referenced** — each artifact carries the ticket ID, each ticket carries the commit SHA. Two authorities, honestly labeled. A fine starting point; a bad resting place.

## Reading the numbers

Prefer what the toolchain already emits — commit timestamps, PR metadata, CI results — over anything self-reported.

Two gauges, and they only mean something together: **how long a change waits between artifacts**, and **how often work gets redone**. Cycle time falling while rework climbs means the process has gotten faster at producing garbage. Watch for that specifically; each skill names the signal worth tracking for its own play.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
