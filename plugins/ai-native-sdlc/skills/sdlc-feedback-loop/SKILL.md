---
name: sdlc-feedback-loop
description: Give the agent a way to check its own work before a human sees it — one-command build and test, a target it can evaluate, a failing test written first for bug fixes, and a visual comparison for UI. Use when setting up a repo for agentic work, when the agent reports done without evidence, when a person is verifying every change by hand, when fixing a bug, when building against a mock, or when asked how to make sessions run longer unattended. Covers the setup, protecting the check from the agent that is fixing the code, and how a final verifier differs from the loop itself.
---

# Let the agent check itself

The highest-leverage thing in this whole set. Without it, the first reliable signal that code works arrives from CI, a reviewer, or production — and when an agent is producing the code, a late signal means a person verifies *everything*. That person is now the constraint, and adding agents makes it worse rather than better.

With it, the session finds and fixes its own mistakes, and what reaches a human has already cleared the bar.

## Four moves

**One command.** If verifying today means running four things and knowing which failures matter, wrap it: `bun run check`, `make verify`. It must exit non-zero on failure — an agent reads the exit code before it reads your prose.

**Show what healthy looks like.** List the commands in `CLAUDE.md` with a line of expected output, so pass and fail are distinguishable without asking.

**Give each task a target the agent can evaluate.** "The three cases in `persistence.test.ts` pass." "The endpoint returns 409 on a duplicate." "The rendering matches the mock at 375px." Not "make it work" — that has no closing condition, so the agent guesses at done.

**Put verification inside the definition of done**, in `CLAUDE.md`:

```markdown
## Before you say it's done
Run `bun run check` and `bun test`, and paste the output.
Both must be clean — a skipped test is a failing test.
If something fails, fix the code. Do not edit the test to make it pass,
and do not delete it. If the test itself is wrong, say so and stop.
```

That last clause matters. Without it, "make the tests pass" has an obvious wrong answer available.

## Bug fixes: test first, in this order

1. Reproduce the bug **as a test**.
2. Run it. Confirm it fails, **and that it fails for the reason you expect** — a test failing for an unrelated reason proves nothing and will pass for the wrong reason later.
3. Commit the test.
4. *Now* fix the code, without touching the test.

A test that existed before the fix and couldn't be edited by whoever wrote the fix is evidence the bug is gone. Anything else is a claim.

## UI: close the loop visually

Give the agent a way to see the result — a browser tool, a screenshot utility — plus the mock, then let it cycle: build, capture, compare, adjust. Two or three rounds is normal and each should visibly improve.

Without it there is no loop on UI work at all, and every round trip costs a person.

## Protect the check

An agent fixing code must not be able to weaken the check on that code. Either a hook blocks edits to test files during a fix (`sdlc-hooks`), which is deterministic and preferred, or review rejects any test change that arrives as part of a fix. Unprotected, this is the single easiest way for green CI to mean nothing.

## The loop vs. a final verifier

Different jobs, both useful:

- **The loop** runs continuously during the work, as many times as it takes.
- **A verifier** (`sdlc-parallel-work`) runs once at the end, in a clean context, so its verdict isn't shaped by the assumptions that produced the code.

The second catches a specific failure: the session that has convinced itself.

## What this gives review

By the time a human looks, the mechanical evidence is attached — the actual test output, the build log, the screenshot. That's what lets a reviewer spend their attention on intent and risk instead of re-running things (`sdlc-pr-review`).

## Worth watching

First-pass CI success on agent-written changes. Then review time per PR, which should fall once the tests are catching what reviewers used to catch by hand.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
