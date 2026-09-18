---
name: sdlc-pr-review
description: Give every PR the same set of agentic review passes — bugs, security, and compliance against spec.md and plan.md — with findings ranked by severity so human attention goes to intent and risk. Use when PRs queue or review quality varies with the reviewer's load, when setting up automated review, when asked to write a REVIEW.md or review policy, when tuning nit volume, when addressing review comments on an agent-authored PR, or when deciding what a human must still approve. Covers the passes, the Important-vs-nit line, the fix loop, and feeding findings back into CLAUDE.md.
---

# AI in the PR review loop

Review capacity used to be planned around human output: a PR waits for a reviewer to read all of it, quality varies with their load, the author chases, the backlog grows. When an agent writes most of the diff, reading every line by hand stops being possible at all.

The answer is not less review. It is **identical passes on every PR, ranked by severity**, with human attention moved up a level — to whether the change does what the plan intended and whether the risk is acceptable.

Review runs in both directions: the agent reviews incoming PRs, and addresses review comments on its own.

## Prerequisites

An up-to-date `CLAUDE.md` (`sdlc-claude-md`); skills, if review passes enforce written policies; `spec.md` and `plan.md` if the compliance pass is to mean anything (`sdlc-spec`, `sdlc-plan`).

## The review policy: `REVIEW.md`

At the repo root, divided into the passes that matter here:

```markdown
# Review instructions

## Passes
Run three passes and tag each finding with its pass:
- Bugs: logic errors, broken edge cases, subtle regressions
- Security: injection risks, authentication gaps, PII in logs
- Compliance: the change matches spec.md, plan.md and our design principles

## What Important means here
Reserve Important for findings that would break behavior, leak data
or breach a policy. Style and naming are nits.

## Cap the nits
Report at most five nits per review; summarize the rest as a count.

## Do not report
Generated files under src/gen/ and anything CI already enforces.
```

Four things every `REVIEW.md` needs: **the passes**, **what Important means**, **a nit cap**, **an exclusion list**. Without the last two, volume drowns signal and reviewers start skimming — which is the failure mode this play exists to prevent.

## The human threshold

**Findings do not approve or block a PR on their own.** Branch protection still requires approval from a code owner. Severity counts are published as a machine-readable tally, so gating merges on them is a separate, deliberate choice.

Separation of duties is what makes this safe: **the agent that wrote the code has no route to approve it.**

## The fix loop

- Tag `@claude` on a review comment and the agent addresses it and pushes the fix. The thread records both the request and the change.
- For PRs the agent opened, let it run the PR to merge: sweep unresolved review comments and failing checks, address them, push, repeat until the PR is green and waiting only on code-owner approval. Wrapping that sweep in a slash command is worth it.

## Findings feed back

**When review flags the same mistake a second time, the correction goes into `CLAUDE.md` as part of that review.** Because review reads `CLAUDE.md`, it is caught from the next PR onward — the loop closes on itself. Review should also flag when a change has left `CLAUDE.md` out of date.

## Tuning

Once a month: rate the findings so the reviewer improves, cap nit volume in `REVIEW.md`, and exclude generated paths and anything CI already enforces. An untuned reviewer trends toward noise.

## Governance

`REVIEW.md` is applied to every PR identically. Findings, fixes, ratings and approvals are logged in the PR history, so **the PR is the audit record**. Approval comes from a human through branch protection, informed by the findings.

## Measuring it

- **Leading** — time to first review (should fall to minutes), and the share of review comments resolved without a human touching the branch.
- **Lagging** — defects and vulnerabilities caught before merge, set against those escaping to production.
