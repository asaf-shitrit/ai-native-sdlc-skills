---
name: sdlc-agent-evals
description: Regression-test the configuration that steers the agent — run an eval suite whenever CLAUDE.md, skills, hooks, subagents, or the model change, and gate the change on the pass rate. Use when editing agent configuration and wondering whether it made things worse, when swapping or upgrading models, when a prompt or skill is rewritten, when asked to set up evals or measure agent quality, or after an incident that should never recur. Covers building the suite from real tasks, the CI workflow, and the incident-to-eval rule.
---

# Continuous evals for agent configuration

Evals are the AI-native equivalent of stage-gate QA. `CLAUDE.md`, skills, hooks and subagent definitions steer the agent as surely as code steers the program — so they deserve the regression testing code gets. When a new model is swapped in or a skill is rewritten, the suite says whether the agent still does the work to the same standard.

## Prerequisites

A tuned `CLAUDE.md` (`sdlc-claude-md`) and a working feedback loop (`sdlc-feedback-loop`) — an eval's checks are the feedback loop's checks, run non-interactively. CI that can run Claude Code headless, and budget for eval runs.

## Building the suite

1. **Collect 20–50 real tasks from recent work**, each with its expected or accepted outcome. Real tasks, not invented ones — invented tasks test the suite, not the agent.
2. **Write each as an eval**: the prompt, plus the checks that define acceptable. Tests pass, lint clean, behavior unchanged, policy followed.
3. **Run it non-interactively in CI** on a schedule and on any change to `CLAUDE.md`, skills, or hooks.
4. **Gate configuration changes on the results.** A skill change that drops the pass rate gets reviewed before it merges.
5. **Every production incident gets an eval**, written by the team that owned the incident, and it stays in the suite as a regression test.

Some teams prefer running offline on a cadence rather than on every change; that is a cost/latency tradeoff, not a correctness one.

## The suite is alive

As models improve, cases that once discriminated stop doing so — everything passes and the suite tells you nothing. Retire dead cases and add new ones from whatever ongoing monitoring surfaces (`sdlc-close-loop`). A suite that has been green for six months is probably not measuring anything.

## The workflow

```yaml
name: Agent evals
on:
  pull_request:
    paths: ['CLAUDE.md', '.claude/**']
  schedule:
    - cron: '0 2 * * *'
jobs:
  evals:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install -g @anthropic-ai/claude-code
      - name: Run eval suite
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          for eval in evals/*.json; do
            claude -p "$(jq -r '.prompt' $eval)" \
              --allowedTools "Read,Edit,Bash(make test)" \
              --output-format json > result.json
            ./evals/check.sh "$eval" result.json
          done
```

Note the `paths` filter — the trigger is a change to the *configuration*, which is the thing under test.

## Governance

The pass-rate threshold is enforced as a merge check. Runs are logged so results can be compared over time. The team that owns the configuration change approves it.

## Measuring it

- **Leading** — eval pass rate over time, and how long a production incident takes to become a permanent eval.
- **Lagging** — regressions caught in CI versus regressions found in production.
