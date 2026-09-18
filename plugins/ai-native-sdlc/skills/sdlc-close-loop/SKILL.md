---
name: sdlc-close-loop
description: Close the SDLC loop — a deterministic detector watches production, invokes the agent when a control band is breached, and what it finds re-enters the pipeline as intent.md. Use when designing autonomous or headless agent workflows, when asked to run Claude non-interactively or from CI/cron/webhook, when setting up monitoring that acts rather than just alerts, when defining autonomy tiers or what an agent may do unsupervised, when post-mortem actions never reach the codebase, or when incidents wait on a person to restart the process. Covers control bands, the tiered response config, rollback, and per-environment autonomy.
---

# Closing the loop

Maintenance is normally reactive: an alert fires at 3 a.m. and is missed, a ticket sits in the backlog, post-mortem actions never reach the codebase because another fire started. Every path needs a person to restart the process.

Closing the loop means a trigger — a breached control band, a ticket, a channel message, a schedule — invokes the agent **with no person in the invocation path**. It diagnoses, acts only through gated routes, and writes what it finds as `intent.md`, which re-enters at Plan. People triage and review that work; they no longer have to start it.

## Prerequisites

The `intent.md` format (`sdlc-intent`) — the loop needs a structured output to restart from. A PR review gate (`sdlc-pr-review`), hooks as an action boundary (`sdlc-hooks`), and **a rollback path that has actually been rehearsed**.

## Detection stays deterministic

⚠️ **No model is involved in detection.** A script watches the metric; the model is invoked only *after* a band is breached, and the tier sets what it may do. Mixing the two makes the trigger unexplainable, which is exactly what you cannot afford at 3 a.m.

1. **Pick one metric with a stable rolling baseline** — CI test failure rate, post-deploy 5xx rate, PR cycle time.
2. **Write the detection script**: typically mean and standard deviation over a rolling window, with rules (Western Electric or similar) so the bands catch slow drift as well as spikes. Version-controlled and unit-tested, like any other code.
3. **Define response tiers in version-controlled config.**
4. **Pick the trigger layer** — a scheduled CI workflow, a webhook from the existing monitoring stack, or a cron job inside the network. The agent runs stateless and non-interactive, so a loop can begin and end without anyone starting it.

### Tiered response

```yaml
metric: ci_test_failure_rate
baseline: rolling_30d
rules: western_electric
tiers:
  1sigma: { action: log }
  2sigma: { action: diagnose,
            tools: "Read,Grep,Bash(gh run view *)" }
  3sigma: { action: propose,
            routes: [pull_request, runbook:rollback-deploy] }
```

Escalating capability, not escalating urgency: log → read-only diagnosis → propose, and even "propose" means **open a PR into the review gate or trigger a pre-approved runbook** — never act directly on production.

## The output is an `intent.md`

The agent writes its diagnosis in the Plan format: the anomaly and its evidence, a proposed outcome, affected systems, open questions. From there the finding goes through the pipeline like anything else — which is the point. There is no separate incident path that skips review.

Then: the service owner or on-call triages the queue. Fix now, schedule, or dismiss — and **dismissals tune the bands**, so noise falls over time rather than being endured.

When a fix ships, **add an eval for the incident** (`sdlc-agent-evals`) so the class is protected against from then on.

## Worked examples

- CI test failure rate breaches 3σ → the agent quarantines the flaky test or opens a revert PR; the review gate decides.
- Post-deploy 5xx rate breaches 3σ with a deployment in the window → the agent triggers the existing rollback pipeline.
- PR cycle time trips a drift rule → the agent writes a report for engineering leadership. The same harness works for process metrics, not just production ones.

## Running the agent headless

- **Start with read-only judgment steps.** `claude -p` in a pipeline job to triage a failed build, summarize a flaky test, or draft a changelog:
  ```yaml
  - name: Triage failed build
    if: failure()
    run: >
      claude -p "Read the build log at out/build.log. Identify the most
      likely cause, say whether the failure looks flaky or real, and write a
      three-line summary for the PR thread." >> triage.md
  ```
- **Add write steps behind the existing gates** — fixing lint, updating generated docs, addressing review comments. Anything written arrives as a PR through branch protection; **the agent has no route to push to main**.
- **Sandbox execution.** Agent jobs run in containers under a network policy with short-lived scoped tokens, holding no production credentials by default.
- **Expose deployment through MCP**, scoped per environment, so deployment powers are an allowlist rather than a shell script with credentials.

## Tier autonomy by environment

- **Development** — the agent deploys freely
- **Staging** — somewhere in the middle
- **Production** — the agent prepares the release; a named release manager authorizes it; a hook enforces the gate

The governing principle: **the agent may act up to the production gate and cannot pass it.**

**Rollback should be the most rehearsed path in the pipeline** — a single command the agent can run, exercised regularly in staging. The 3σ tier calls it, so it has to be proven *before* it is needed.

## Work arriving through other channels

Incidents also arrive as a 10pm message in a chat channel. An agent present in the channel under its own identity gives every incident a first responder, and the response becomes part of the record: request, diagnosis, human authorization and fix all stay where the incident was handled. Small bounded fixes arrive as a PR through the review gate; anything larger is written up as `intent.md` and starts at Plan. The post-mortem goes to a version-controlled lessons file that future investigations read.

## Governance

Tier boundaries are enforced from version-controlled config, with permissions denying production access. Invocations, findings and triage decisions are logged with timestamps. A service owner triages and approves; resulting changes go through the normal PR gate; the runbooks the agent may trigger were approved in advance.

## Measuring it

- **Leading** — time from band breach to an `intent.md` in the triage queue, against the old time from incident to post-mortem action.
- **Lagging** — share of findings that become merged fixes, and repeat incidents of the same class, which should fall as fixes add cases to the eval suite.
