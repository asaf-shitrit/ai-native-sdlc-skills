---
name: sdlc-hooks
description: Make a rule deterministic with hooks — block edits to protected paths, run formatters after edits, keep credentials out of diffs, and pause an action until a named human approves. Use when a policy must hold without exception, when a skill or CLAUDE.md instruction keeps getting ignored, when an approval gate must survive automation (release authorization, change sign-off, protected paths), when asked to configure .claude/settings.json, or when deciding between a skill and a hook. Covers allow/ask/block, the settings shape, an example gate script, and where hooks belong in a session.
---

# Hooks: the deterministic layer

A skill is advisory — it makes the agent *likely* to comply. A hook is deterministic: it runs on every matching action, for everyone, every time.

**Back any skill whose policy has to hold without exception.** The skill makes violations rare; the hook makes them close to impossible.

## Three verdicts

- **Allow** — pre-approve the safe inner loop, so the deny rules do not turn into prompt fatigue
- **Block** — refuse the action outright (exit 2; the message goes back to the agent)
- **Ask** — pause until a specific person approves

## Build-time guardrails vs. approval gates

Most of an agent's actions are file edits and shell commands during implementation, so that is where hooks fire most often.

**Guardrails** (no human involved — use freely):
- Block edits to protected paths: generated classes, a frozen package, migrations or infra without a change ticket
- Run the formatter and linter after file edits, so drift never accumulates
- Keep credentials out of the diff
- Block edits to test files during a fix task (`sdlc-feedback-loop`)
- Keep `plan.md` in sync with the diff (`sdlc-plan`)

Guardrails must be **fast and scoped to the file that changed**. Heavier checks — the full test suite — belong at the commit or the PR, not on every edit.

**Approval gates** (a human must say yes):
- Release authorization for a production deploy
- Change-management sign-off
- Anything where the organization requires a named approver

⚠️ **An approval prompt during the build puts a person back on the critical path of every session running in parallel.** Keep gates at the release boundary; keep the build phase to allow/block only.

## Shape

`.claude/settings.json`, checked into git so the team shares it:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/production-gate.sh" }
        ]
      }
    ]
  }
}
```

The gate itself, `.claude/hooks/production-gate.sh`:

```bash
#!/bin/bash
# Production deploys require a named release authorization
cmd=$(jq -r '.tool_input.command' < /dev/stdin)
if [[ "$cmd" == *"deploy"* && "$cmd" == *"production"* ]]; then
  if [ -z "$RELEASE_APPROVAL" ]; then
    echo "Production deploys need a release authorization." >&2
    exit 2   # exit 2 blocks the action; the message goes to Claude
  fi
fi
exit 0
```

## A block must explain itself

When a hook stops an action, the reason **and the route to approval** appear in the agent's output. A bare "blocked" costs a human round trip to decode; "needs release authorization, ask the release manager" does not. This is the difference between a gate people route around and one they use.

## Setting the gates

1. List the human approval gates that must survive — with whoever owns change management and compliance, if that applies.
2. Express each as a hook: a script that runs before the action and returns allow, ask, or block.
3. Team hooks go in `.claude/settings.json` in git. Gates that individual engineers must not be able to switch off go in managed settings owned by the platform or IT admin.

## Governance

Hooks *are* the approval gates. The condition is enforced every time, for everyone. Allow and block decisions are logged with a timestamp. The gate also defines what counts as approval — an approved change ticket, the release manager's sign-off.

Related deterministic controls worth pairing with hooks: permission allow/deny rules, and OS-level sandboxing for filesystem and network isolation (a tool-level deny on web fetching does not stop a shell command reaching the network). See `code.claude.com/docs/en/settings`, `/permissions`, `/sandboxing`.

## Measuring it

- **Leading** — time spent waiting at each approval gate. Every hook decision carries a timestamp and an allow/block verdict, so the wait is visible per gate.
- **Lagging** — gate violations reaching production, before and after.

---

*Distilled from Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) by Louis Claxton (August 2026), which is the canonical source. This is an unofficial repackaging into skill form; not affiliated with or endorsed by Anthropic.*
