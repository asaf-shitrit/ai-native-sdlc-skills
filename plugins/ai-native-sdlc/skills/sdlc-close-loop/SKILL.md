---
name: sdlc-close-loop
description: Close the lifecycle loop — a deterministic detector watches production, invokes the agent when a threshold is crossed, and whatever it finds re-enters the pipeline as a new intent. Use when designing autonomous or headless agent workflows, when asked to run the agent non-interactively from CI, cron or a webhook, when monitoring should act rather than only alert, when defining how much autonomy an agent gets per environment, when post-mortem actions never reach the codebase, or when incidents wait on a person before anything starts. Covers deterministic detection, tiered response, rollback, and the boundary the agent cannot cross.
---

# Closing the loop

Maintenance is the stage that stays manual longest. An alert fires overnight and is missed. A ticket sits until someone has capacity. Post-mortem actions never land because the next incident arrives first. Every path needs a person to restart it, and people are the scarce thing.

Closing the loop means a trigger — a threshold crossed, a ticket filed, a message posted, a schedule elapsing — starts the work **with nobody in the invocation path**. The agent investigates, acts only through routes you've approved, and writes what it found as a new intent. People triage and review; they no longer have to notice and begin.

## What has to exist first

An intent format (`sdlc-intent`), because the loop needs a structured way to hand work back. A review gate (`sdlc-pr-review`) and hooks as an action boundary (`sdlc-hooks`), because this is where an agent runs unsupervised. And **a rollback path you have actually rehearsed** — the highest tier calls it, so it cannot be a runbook nobody has executed.

## Detection stays deterministic

⚠️ **No model participates in detection.** A script decides whether something is wrong; the agent is invoked only after that decision, and its tier governs what it may then do.

This is not a performance concern. A trigger you can't explain is a trigger you can't trust at 4am, and the first false alarm that wakes someone will end the experiment.

Pick **one** metric with a baseline stable enough to have a normal range — test failure rate, post-deploy error rate, queue depth, cycle time. Compute the band over a rolling window; use rules that catch sustained drift and not only spikes, since the slow version is the one people miss. Version the script and unit-test it like anything else, because a broken detector fails silently.

## Tiered response

Capability escalates with severity, and the top tier still stops short of production:

```yaml
signal: post_deploy_error_rate
window: rolling_14d
detector: ewma            # sustained drift, not just spikes

tiers:
  warn:
    at: 2_sigma
    do: record            # log it; build the baseline for later
  investigate:
    at: 3_sigma
    do: diagnose
    allow: [read, search, "logs:query", "ci:read"]
    output: written_finding
  respond:
    at: 3_sigma_sustained
    do: propose
    via:
      - open_pull_request
      - runbook: rollback_last_deploy
    never: [direct_production_write, force_merge, credential_access]
```

The `never` list is the point. **The agent may work right up to the production gate and cannot pass it** — everything it produces arrives as a proposal through a route a human already approved.

## The output is an intent

The finding is written in the normal intent format: what moved, the evidence, what should change, what's affected, what's still unknown. From there it goes through the pipeline like any other work.

That's deliberate. **There is no express lane for incident-driven changes** — the path that skips review is exactly the path that produces the next incident.

Someone triages the queue: fix now, schedule, or dismiss. **Dismissals tune the bands**, so noise decreases over time instead of being endured until people mute the channel.

When a fix ships, add an eval for it (`sdlc-agent-evals`) so that failure mode is covered permanently.

## Running headless

Start read-only. Judgment steps that don't write anything are where this earns trust:

```yaml
  - name: Explain the failure
    if: failure()
    run: |
      claude -p "$(cat <<'PROMPT'
      Read ci-output.log. Say what failed, whether it looks like a real
      regression or infrastructure flake, and what evidence points that
      way. Three sentences. If you cannot tell, say so.
      PROMPT
      )" --allowedTools "Read,Grep" >> "$GITHUB_STEP_SUMMARY"
```

Then add write steps **behind the gates you already have** — fixing lint, regenerating docs, addressing review comments. Everything arrives as a PR through branch protection; there is no direct path to the default branch.

Run these jobs in a sandbox with short-lived scoped credentials and no standing production access. Expose operational actions as scoped tools rather than shell commands with credentials, so what the agent can do is an allowlist rather than whatever the shell permits.

## Autonomy by environment

| Environment | Agent may |
|---|---|
| Development | Act freely; mistakes are cheap and instructive |
| Staging | Deploy and exercise, including rehearsing rollback |
| Production | Prepare and propose. A named person authorizes; a hook enforces it |

**Rollback should be the best-rehearsed path you have** — one command, exercised regularly in staging, proven long before the tier that calls it ever fires.

## Work arriving from elsewhere

Incidents also show up as a message at 10pm. An agent present in that channel gives every incident a first responder, and keeps the record where the work happened: what was asked, what was found, who authorized the fix. Small bounded fixes become PRs through the normal gate; anything larger becomes an intent. The write-up goes somewhere version-controlled that future investigations can actually read.

## Worth watching

Time from threshold crossed to a triageable finding, against how long it used to take for anyone to notice. Then: what share of findings become merged fixes — and repeat incidents of the same class, which should decline as fixes accumulate in the eval suite. If findings pile up untriaged, the bands are too loose and the loop is generating work rather than absorbing it.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
