---
name: sdlc-agent-evals
description: Regression-test the configuration that steers the agent — run a suite of real tasks whenever CLAUDE.md, skills, hooks, subagent definitions or the model change, and gate the change on the result. Use when editing agent configuration and wondering whether it helped, when upgrading or switching models, when a skill or prompt is rewritten, when asked to set up evals or measure agent quality, or after an incident that must not recur. Covers assembling the suite from real work, wiring it into CI, and why a suite that always passes has stopped working.
---

# Test the configuration, not just the code

`CLAUDE.md`, skills, hooks and subagent definitions steer the agent as surely as source code steers the program. They're edited by hand, they interact, and their effects are invisible until something goes wrong in a diff nobody expected. That makes them exactly the kind of thing regression tests exist for.

A suite answers one question: **after this change, does the agent still do the work to the standard it did before?**

## What you need first

Something worth measuring — a real `CLAUDE.md` (`sdlc-claude-md`) and a working verification story (`sdlc-feedback-loop`), because an eval's checks are that same verification run headlessly. Plus CI that can run the agent non-interactively, and budget for the runs.

## Assembling the suite

**Take 20–50 tasks from work you actually did**, each with the outcome you accepted at the time. Real tasks. Invented ones test your imagination, and they're systematically easier than reality because you wrote them knowing the answer.

**Each eval is a prompt plus its checks** — tests green, types clean, behavior unchanged, the relevant rule followed. The checks have to be mechanical; a human grading 40 outputs per config change will not keep doing it.

**Every incident earns an eval**, written by whoever owned the incident, and it stays in the suite permanently. This is how the suite gets good: it accumulates the specific ways this codebase goes wrong.

**Gate config changes on the result.** A skill edit that drops the pass rate gets looked at before it merges — which is the entire point, since the alternative is discovering it across everyone's sessions next week.

```yaml
name: agent-evals

on:
  pull_request:
    paths:
      - 'CLAUDE.md'
      - '.claude/**'
      - 'evals/**'
  schedule:
    - cron: '17 4 * * 1'        # weekly, to catch model-side drift
  workflow_dispatch:

jobs:
  suite:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - run: npm i -g @anthropic-ai/claude-code
      - name: Run suite
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: ./evals/run.sh --report results.json
      - name: Compare against baseline
        run: ./evals/gate.sh results.json --min-pass 0.9
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: eval-results
          path: results.json
```

Note the `paths` filter: the trigger is a change to the **configuration**, because the configuration is what's under test. The weekly run catches the other source of drift — the model changing underneath you while your files sit still.

If per-change runs are too slow or too expensive, run on a cadence instead. That's a latency trade, not a correctness one.

## A green suite is a warning sign

Cases that discriminated a year ago stop discriminating as models improve. Everything passes, the number looks great, and the suite is measuring nothing.

So prune dead cases and keep adding from whatever real work and monitoring surface (`sdlc-close-loop`). **A suite that hasn't failed in months is not a healthy suite — it's an unmaintained one**, and it will be green through the change that breaks you.

## Worth watching

Pass rate over time, read against what changed. And the lag from an incident to its eval existing — if that's measured in months, the suite isn't really absorbing what you learn.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
