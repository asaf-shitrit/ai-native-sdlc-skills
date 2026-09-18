---
name: sdlc-hooks
description: Make a rule deterministic with hooks — block writes to protected paths, run a formatter after edits, keep secrets out of a diff, or hold an action until a named person approves. Use when a rule must hold every time, when a documented convention keeps getting ignored, when an approval must survive automation (releases, migrations, infrastructure, protected paths), when asked to configure settings.json or write a hook script, or when deciding whether something should be a skill or a hook. Covers allow/ask/block, where each belongs in a session, and why the block message matters as much as the block.
---

# Hooks: the part that actually holds

A documented rule is advisory — it makes compliance likely. A hook is deterministic: it runs on every matching action, for everyone, whether or not anyone remembered it.

**Put a hook behind any rule whose violation you can't live with.** Documentation makes violations rare; the hook makes them not happen. The two together is the working combination — a hook with no explanation is obeyed but not understood, and documentation with no hook is understood but not obeyed.

## Three verdicts

- **allow** — pre-approve the safe inner loop, so routine commands don't generate a stream of prompts. Without this the rest becomes noise people click through.
- **block** — refuse outright. Exit non-zero; the message goes back to the agent.
- **ask** — hold until a specific person approves.

## Where each belongs

Most agent actions are file edits and shell commands during implementation, so that's where hooks fire most.

**Guardrails — no human in the loop. Use these freely:**

| Guard | Why |
|---|---|
| Block writes to generated output, vendored code, frozen packages | The agent can't know these are off-limits by looking at them |
| Format and lint after an edit | Drift never accumulates; review never spends attention on it |
| Reject secrets in a staged diff | Cheapest possible place to catch it |
| Block test edits during a fix task | Protects the check from whoever is fixing the thing it checks (`sdlc-feedback-loop`) |
| Require the plan to change when the diff leaves it | Keeps the artifact honest (`sdlc-plan`) |

Guardrails must be **fast and scoped to what changed**. A hook runs on every matching action, so a slow one taxes the whole session. The full test suite belongs at commit or CI, not on every file write.

**Gates — a person must say yes:** production releases, destructive data operations, infrastructure changes, anything with a named approver in a process you can't unilaterally change.

⚠️ **Keep gates at the boundary, not in the build.** An approval prompt mid-implementation blocks every parallel session at once and trains people to approve reflexively — which costs you the gate you were trying to build.

## Shape

Project hooks live in `.claude/settings.json`, committed, so the team shares one definition:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/guard-release.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/usr/bin/env bash
# Hold anything that ships to production until a release is authorized.
set -euo pipefail

cmd=$(jq -r '.tool_input.command // ""')

if grep -qE '(^|[[:space:]])(fly deploy|kubectl apply|terraform apply)' <<<"$cmd" \
   && grep -qE '(prod|production)' <<<"$cmd"; then

  if [[ -z "${RELEASE_TICKET:-}" ]]; then
    cat >&2 <<'MSG'
Blocked: production changes need an authorized release.

  Why:  every production change is traceable to a named approver.
  Fix:  get sign-off, then re-run with RELEASE_TICKET=<id> set.
  Who:  whoever is on release duty this week.
MSG
    exit 2
  fi
fi

exit 0
```

Gates that individuals must not be able to disable belong in settings owned by whoever administers the machines, not in the repo — anything in the repo can be edited by anyone who can edit the repo.

## Make the block teach

When a hook stops something, the output must carry **why, and how to proceed legitimately**. A bare "blocked" costs a human round trip to decode, every time, forever. The version above costs nothing and routes the person correctly.

This is the difference between a gate people use and a gate people work around.

## Adjacent controls

Hooks govern the agent's actions. Two other layers matter and neither substitutes for the other: **permission rules** on which tools and commands are available at all, and **sandboxing** at the OS level for filesystem and network isolation. Denying a network tool doesn't stop a shell command reaching the network — only the sandbox does. See `code.claude.com/docs/en/hooks`, `/permissions`, `/sandboxing`.

## Worth watching

Time spent waiting at each gate — every decision is timestamped with its verdict, so a gate that's become a queue is visible rather than folklore. And whether the thing the gate exists to prevent still reaches production.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
