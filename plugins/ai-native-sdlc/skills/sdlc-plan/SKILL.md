---
name: sdlc-plan
description: Work from a written, interrogated, committed plan.md before any code is generated — files that change, order of work, risks, and the proof that it worked. Use before implementing anything non-trivial, when starting in plan mode, when asked "how would you do this", when a change touches more than one file or has a blast radius, and when deciding whether a session can safely run on auto-accept. Covers the plan template, the questions that make a plan worth having, keeping the diff and the plan in sync, and when auto mode is appropriate.
---

# Plan before building

Third artifact in the chain (see `ai-native-sdlc`). Traditionally, how a change would be made lived in the engineer's head; the first thing a reviewer saw was the finished diff, and by then rework was expensive. Plan mode moves design review to when changing course is still a matter of editing a document.

## Prerequisites

`intent.md` and/or `spec.md` if they exist; `CLAUDE.md` helps a lot (`sdlc-claude-md`).

## The pass

1. **Start in plan mode.** The agent reads the codebase and cannot edit it. That restriction is the control — it is enforced by the harness, not by good intentions.
2. **Ask for a plan that names**: the files that change, the order of the work, and the tests that prove it. Hand it `intent.md` and `spec.md`.
3. **Interrogate the plan.** This is where the value is, and it is the step people skip:
   - What could this change break?
   - Which step is the riskiest?
   - What did you consider and reject, and why?
   - What do you not know yet?
4. **Iterate until an engineer who never saw the conversation could implement the change from the plan alone.** That is the bar.
5. **Commit the approved plan as `plan.md`.** It joins the audit trail, and PR review (`sdlc-pr-review`) checks the eventual diff against it.
6. **Accept and implement.** With a solid plan, implementation is often a single pass.
7. **When implementation departs from the plan, update `plan.md` in the same commit.** A hook can enforce that the two stay in sync (`sdlc-hooks`).

## Template

```markdown
# Plan: claims status self-service (from intent.md 2026-06-02)

## Files that change
portal/src/claims/StatusPanel.tsx (new), claims-api/routes/status.py,
claims-api/tests/test_status.py

## Order of work
1. Add the status endpoint behind existing auth.
2. Panel against the endpoint.
3. Wire into the portal nav.

## Risks
The claims-core API rate-limits at 50 rps; the panel must cache.

## Proof
test_status.py covers the four claim states; screenshot matches the
approved mock.
```

**Proof** is not optional. A plan without a stated, checkable proof condition cannot close its own loop (`sdlc-feedback-loop`), which means a human has to check everything by hand.

## Auto mode

Once the plan is approved, the agent can apply each change without a per-edit prompt. Auto-accept is the right default for routine work when **all** of these hold:

- A tight spec and an approved plan
- Small blast radius
- Code the tests already cover
- Guardrails are in place: a tuned `CLAUDE.md`, skills encoding policy, hooks blocking unsafe actions, and a test suite the agent can run

The shift auto mode buys is from *watching the agent edit* to *reviewing artifacts after longer autonomous sessions*. It is also what makes parallelism worthwhile (`sdlc-parallel-work`) and is a precondition for running the loop autonomously (`sdlc-close-loop`).

If the guardrails are not in place, auto mode just means finding out later.

## Governance

Plan mode enforces design-before-code by construction: no file can be edited until the plan is accepted. The plan and its revisions are logged along with who accepted it. Routine changes are approved by the engineer; higher-risk classes go to a tech lead or architect.

## Measuring it

- **Leading** — share of changes that merge from the first implementation pass; time from plan approval to merged PR.
- **Lagging** — rework cycles per change (PR metadata), and how often the merged diff still matches the committed `plan.md`.

---

*Distilled from Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) by Louis Claxton (August 2026), which is the canonical source. This is an unofficial repackaging into skill form; not affiliated with or endorsed by Anthropic.*
