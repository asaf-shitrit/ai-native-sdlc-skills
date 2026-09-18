---
name: sdlc-parallel-work
description: Run several streams of work at once with git worktrees, and factor recurring jobs into subagents with their own context and tool limits. Use when there is more work in flight than one session can carry, when asked to parallelize or run tasks concurrently, when a job recurs across tasks (verifying the app runs, simplifying after implementation, exploring the codebase without flooding context), when defining .claude/agents, or when deciding how many sessions one person can actually supervise. Covers splitting work so sessions do not collide, the subagent definition, and the real ceiling on parallelism.
---

# Parallel sessions and subagents

One person can drive several streams of work at once. The job shifts from writing to **steering and reviewing** — and eventually to building and monitoring loops (`sdlc-close-loop`).

## Two different things

- **A parallel session** is another full Claude Code instance, on a separate task, in its own git worktree. Sessions know nothing about each other; the person steering them is the only thing they share.
- **A subagent** runs *inside* one session as a scoped helper with its own context window and tool limits.

Parallel sessions raise the number of tasks in flight. Subagents keep each session focused on its own task. They compose; they do not substitute for each other.

## Prerequisites

- **`CLAUDE.md`** (`sdlc-claude-md`) — every session reads it, so it is what keeps parallel work consistent
- **A feedback loop** (`sdlc-feedback-loop`) — the real enabler. A session that can verify its own work needs far less supervision, which is the only reason several can run at once
- **Permission settings tuned** so sessions are not sitting on approval prompts for commands the team considers safe

## Splitting the work

1. **Split into tasks that touch different files.** Use the plan (`sdlc-plan`) to see where the work is genuinely independent.
2. **Tasks that share files run in a single session, one after another.** Do not fight merge conflicts you chose to create.
3. **Each parallel task gets its own worktree** — a separate checkout on its own branch, so sessions cannot collide on files:
   ```
   claude --worktree feature-auth       # one terminal
   claude --worktree fix-rate-limit     # another
   ```
4. **Start with two or three.** The practical ceiling is not machine capacity — it is **how many streams one person can review properly**. Add sessions only while review is keeping up. Past that point you are manufacturing unreviewed diffs.

## Subagents

Turn repeated jobs into subagents: markdown files in `.claude/agents/`, each with a name, a description of when to use it, and the tools it may touch. Check them into git so the team shares them.

Ones that consistently earn their place:

- **A verifier** — runs the app and checks behavior in a fresh context, so the verdict is not colored by the assumptions that produced the code
- **A simplifier** — strips needless complexity after the main agent finishes
- **A researcher** — explores the codebase and reports back without flooding the main context

```markdown
---
name: verifier
description: Runs the app and checks the change works before the session
  reports done
tools: Bash, Read
---

Start the app with make run. Exercise the changed behavior and the two
nearest neighboring flows. Report what you ran, what you saw, and any
behavior that does not match plan.md. Do not fix anything; report only.
```

The last line matters: **report only**. A verifier that fixes what it finds has just become the author, and the independent check is gone.

## Governance

More sessions means more output, so the controls must come from **configuration in the repo** rather than from watching. Hooks and permission settings there apply to every session automatically (`sdlc-hooks`). What a session does is logged and attributed to the person who ran it.

## Measuring it

- **Leading** — concurrent sessions per person *while review quality holds*, and the share of the day spent steering rather than waiting.
- **Lagging** — changes merged per person per week, read **alongside** the rework rate. Either number alone is misleading.
