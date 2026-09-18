---
name: sdlc-pr-review
description: Give every pull request the same review passes — correctness, security, and whether the change matches what was agreed — with findings ranked so human attention lands on intent and risk. Use when PRs queue or review depth varies by who picks them up, when setting up automated review, when asked to write a review policy, when review comments are drowning in style nits, when addressing feedback on an agent-authored PR, or when deciding what a human must still approve. Covers the policy file, keeping severity meaningful, and routing repeat findings back into the repo's context.
---

# Review that scales with output

Review capacity was sized for human authorship. One reviewer reads the whole diff, depth varies with how busy they are, the author chases, the queue grows. Once an agent writes most of the diff, reading every line stops being possible — and "review it anyway" quietly becomes skimming with a rubber stamp on the end.

The fix isn't less review. It's **uniform passes on every PR, findings ranked by severity, and human attention moved up a level** — to whether this change does what was agreed and whether the risk is acceptable.

It runs both directions: the agent reviews incoming changes, and addresses findings on its own.

## What you need

A current `CLAUDE.md` (`sdlc-claude-md`) — review reads it, which is what makes the feedback loop below work. Encoded rules where policy is involved. And a committed spec and plan if you want the third pass to mean anything (`sdlc-spec`, `sdlc-plan`).

## The policy file

Committed at the repo root, so review depth is a property of the repo rather than of whoever opened the PR.

```markdown
# Review policy

## Passes
Tag every finding with the pass that produced it.

1. Correctness — logic errors, unhandled edge cases, broken
   invariants, concurrency and ordering mistakes, error paths that
   swallow failures.
2. Security — injection, missing authorization, secrets or personal
   data in logs and error responses, unsafe deserialization.
3. Agreement — does the change do what spec.md described and what
   plan.md said it would? Call out anything implemented that nobody
   asked for.

## Severity
Blocking     breaks behavior, loses or exposes data, violates a stated rule
Worth fixing real, but the change is still safe to ship
Nit          style, naming, preference

Blocking is for the first definition only. If everything is blocking,
nothing is.

## Volume
At most five nits, then a count. Prefer one accurate finding to four
speculative ones — a reviewer who learns the findings are noisy stops
reading them, and then this whole thing is decoration.

## Out of scope
Generated code, vendored dependencies, lockfiles, and anything the
type checker or linter already enforces in CI.
```

Four things it has to answer: **which passes**, **what blocking means**, **how much noise is tolerated**, **what to ignore.** Miss the last two and volume drowns the signal within a week.

## What stays human

**Findings don't approve or block on their own.** Branch protection still requires a person. Gating merges on severity counts is a separate decision you make deliberately, not a default you inherit.

The property that makes this safe is simple: **whatever wrote the code cannot approve it.** That holds however much of the review is automated.

## The fix loop

Tag the agent on a finding and it pushes a fix; the thread keeps both the request and the change, so the reasoning survives.

For PRs it opened, let it run the loop to completion — sweep unresolved comments and failing checks, fix, push, repeat until the PR is green and waiting only on human approval. Worth wrapping in a command once you've done it twice by hand.

## Findings feed the repo

**A finding raised twice is a documentation bug, not a review finding.** Put the correction in `CLAUDE.md` as part of that review. Since review reads `CLAUDE.md`, it's caught from the next PR onward and stops consuming attention.

Review should also flag when a change has made the existing context wrong — that file goes stale silently, and this is the only routine moment anyone notices.

## Keep it tuned

Periodically: rate findings so the useful ones are reinforced, tighten the nit budget, extend the exclusions. An untuned reviewer trends toward volume, and volume is how this gets switched off.

## Worth watching

Time to first review, which should be minutes. The share of findings resolved without a person touching the branch. Then the one that matters: defects caught before merge versus defects found in production.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
