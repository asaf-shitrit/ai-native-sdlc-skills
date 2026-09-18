---
name: sdlc-feedback-loop
description: Give the agent a way to check its own work before a human sees it — one-command build/test/lint, a quantifiable target, failing-test-first for bug fixes, and a visual check for UI. Use when setting up a repo for agentic work, when the agent reports "done" without evidence, when a human is having to verify every change by hand, when fixing a bug, when implementing against a mock or design, or when asked how to make sessions more autonomous. Covers the CLAUDE.md verification block, protecting the loop from the agent that fixes the code, and verifier subagents.
---

# Give the agent a feedback loop

The highest-leverage play in the whole lifecycle. Without it, the signal that code works arrives late — CI minutes later, a tester days later, production weeks later — and when an agent produces the code, a late signal means a person must check *all* of its output. That person becomes the bottleneck.

With it, the session runs the check, fixes its own mistakes, and what reaches the human has already passed.

## Setup

1. **Collapse verification into one command.** If checking the work today takes a sequence of commands and some environment knowledge, wrap it: `make test`, `npm test`. It must exit non-zero on failure.
2. **List the commands in `CLAUDE.md` with an example of healthy output**, so the agent can tell pass from fail without asking.
3. **State a quantifiable target** per task, so the agent can check its own work: "all tests in `test_status.py` pass", "the screenshot matches the attached mock", "the endpoint returns 200 with the new field". A target like "make it work" cannot close a loop.
4. **Make verification part of "done"** in `CLAUDE.md`: run the checks before reporting a task complete, and show the output.

### The CLAUDE.md block

```markdown
## Verifying your work
- Build: make build (must finish with "Build succeeded")
- Test: make test (all green; never skip or delete a failing test)
- Lint: make lint (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

## Bug fixes: failing test first

The sequence matters:

1. Ask the agent to **reproduce the bug as a test**.
2. **Run it and confirm it fails for the reason you expect.** A test that fails for the wrong reason proves nothing.
3. **Commit that test.**
4. *Only then* ask for the fix, **without editing the test**.

A test that existed before the fix, and that the agent could not rewrite, is proof the bug is gone. Everything else is an assertion.

## UI work: close the loop visually

Give the agent a browser or screenshot tool and the mock, then let it iterate: implement → screenshot → compare → adjust. Two or three rounds is normal, and each round should be visibly better. Without a visual check, UI work has no loop at all and every round trip costs a human.

## Protect the loop

An agent fixing code must not be able to weaken the check on that code. Either:

- **A hook that blocks edits to test files during a fix task** (`sdlc-hooks`) — deterministic, preferred; or
- **Check the diff in review** and reject any change that touches a test as part of a fix.

Left unprotected, "make the tests pass" has an easy wrong answer.

## Feedback loop vs. verifier subagent

They are different things and both are useful:

- **The feedback loop** runs *throughout* the task, as many times as the work needs.
- **A verifier subagent** (`sdlc-parallel-work`) runs *once*, in a fresh context window, after the session believes it is done — so the verdict is not colored by the assumptions that produced the code.

## Governance

- **Enforced** — verification before "done"; no agent edits to test files during a fix. Both as hooks where the guarantee matters.
- **Evidence** — the literal output of `make test`, the build log, the screenshot diff. It comes from the toolchain, not from the agent's summary.
- **Logged** — in the session transcript, and in the PR's check run where the reviewer and any later auditor both see it.
- **Approved by** — the code owner reviewing the PR, who can concentrate on intent and risk because the mechanical evidence is already attached.

## Measuring it

- **Leading** — first-pass CI success rate for agent-written changes.
- **Lagging** — review time per PR (should fall once tests catch what reviewers used to catch), and change failure rate from the incident tracker.
