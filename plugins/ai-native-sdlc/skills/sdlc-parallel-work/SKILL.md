---
name: sdlc-parallel-work
description: Run several streams of work at once using isolated worktrees, and factor recurring jobs into subagents with their own context and tool limits. Use when there is more work in flight than one session can hold, when asked to parallelize or run tasks concurrently, when the same job recurs across tasks (verifying the app runs, simplifying after implementation, exploring a codebase without flooding context), when writing subagent definitions, or when deciding how many sessions one person can genuinely supervise. Covers splitting work so sessions do not collide, what makes a good subagent, and the real ceiling.
---

# Several streams at once

## Two different tools

**A session** is a full instance working one task in its own checkout. Sessions share nothing and know nothing about each other — the person steering them is the only connection.

**A subagent** runs inside a session as a scoped helper with its own context window and its own tool access.

Sessions increase how much is in flight. Subagents keep one session from drowning in its own context. They compose; neither substitutes for the other.

## What has to be true first

**A real `CLAUDE.md`** (`sdlc-claude-md`), because it's the only thing keeping independent sessions consistent with each other.

**A working feedback loop** (`sdlc-feedback-loop`) — this is the actual prerequisite. A session that verifies itself needs supervision at the end; one that can't needs supervision throughout, and you cannot supervise three of those at once. Everything here depends on that.

**Permissions tuned** so sessions aren't parked on prompts for commands you consider routine. Three blocked sessions are worse than one unblocked one.

## Splitting the work

Split along **files**, not features. Use the plan (`sdlc-plan`) to see where the work is genuinely independent. Tasks that touch the same files go in one session, sequentially — parallelizing them just manufactures merge conflicts you then resolve by hand.

Give each task its own worktree, so sessions can't collide:

```bash
git worktree add ../app-draft-persistence -b draft-persistence
git worktree add ../app-webhook-retry     -b webhook-retry
# then start a session in each
```

**Start with two.** The ceiling is not your machine — it's **how many streams one person can review properly**. Add a third only once review is comfortably keeping up. Past that point the output is unreviewed diffs, which is negative value: someone still has to read them, later, with less context, under more pressure.

## Subagents

A job earns a subagent when it recurs across tasks and benefits from a clean context. The three that consistently pay:

- **A verifier** — runs the thing and checks behavior, in a fresh context, so its verdict isn't shaped by the reasoning that produced the code
- **A simplifier** — removes accidental complexity once the work is correct
- **An explorer** — answers a question about the codebase and reports back, without pulling the whole search into the main context

Definitions live in `.claude/agents/`, committed:

```markdown
---
name: verifier
description: Independently confirms a finished change behaves correctly.
  Use when a session believes its work is complete.
tools: Bash, Read, Grep
---

Read plan.md for what this change was supposed to do.

Start the app (`bun run dev`). Exercise the changed behavior, then the
two flows nearest it that were working before — regressions show up
next door more often than in the change itself.

Report: what you ran, what you observed, and anything that does not
match plan.md. Include the exact commands.

Do not fix anything. Do not edit files. If something is broken, say
what and stop.
```

**"Do not fix anything" is load-bearing.** A verifier that repairs what it finds has become the author, and the independent check you built is gone — with the added cost that nobody knows it's gone.

## Where control comes from

More concurrency means less watching, so the controls have to be in the repo rather than in your attention. Hooks and permission rules apply to every session automatically (`sdlc-hooks`); session activity is logged and attributed. Anything you enforce by noticing does not survive this play.

## Worth watching

Concurrent sessions **while review quality holds** — that qualifier is the whole measurement. And throughput read alongside rework, never on its own; merged-changes-per-week climbing while rework climbs faster is the failure this play invites.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
