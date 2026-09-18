# AI-Native SDLC skills

Eleven [Claude Code skills](https://code.claude.com/docs/en/skills) that turn the AI-native software development lifecycle into something an agent actually follows: a chain of committed artifacts, from a raw idea through to production monitoring that feeds the next idea back in.

The premise, in one line: **code is no longer the bottleneck — the human-speed steps to the left and right of it are.** Plan, review, test and deploy become the constraint once an agent writes most of the diff, and controls designed around a person reading every line stop matching reality. The answer isn't less process. It's the same control objectives with new enforcement.

## The artifact chain

```
idea / ticket / alert
   ↓  intent.md      what is wanted, why, under what constraints   [sdlc-intent]
   ↓  spec.md        requirements + design, concerns flagged       [sdlc-spec]
   ↓  plan.md        files, order, risks, proof                    [sdlc-plan]
   ↓  diff + tests   implementation that verified itself           [sdlc-feedback-loop]
   ↓  PR + findings  agentic review passes, human judges risk      [sdlc-pr-review]
   ↓  production     control bands watch it                        [sdlc-close-loop]
   ↺  breach → new intent.md
```

Every stage ends by committing something the next stage reads, so the chain of commits *is* the audit trail: who asked for what, the agent produced, who approved it. Humans stay accountable for every judgment call — what changes is which artifact they read.

## The skills

| Skill | What it carries |
|---|---|
| **`ai-native-sdlc`** | Router. The artifact chain, which skill fits which situation, adoption order by dependency, and how to read leading vs lagging metrics as a pair |
| `sdlc-intent` | Capture an idea, ticket or alert as `intent.md` — brainstorm-first interview, the template, where intent lives, when not to bother |
| `sdlc-spec` | Requirements and design in one session; flagged contradictions are the deliverable, not the prose |
| `sdlc-plan` | Files / order / risks / **proof** before code, the four questions that make a plan worth having, when auto mode is earned |
| `sdlc-claude-md` | Institutional knowledge as files — the CLAUDE.md-vs-skill-vs-prompt boundary, one-page rule, twice-wrong rule |
| `sdlc-feedback-loop` | One-command verification, failing-test-first, visual loops, and protecting the check from the agent that fixes the code |
| `sdlc-agent-evals` | Regression-test the configuration that steers the agent; every incident becomes a permanent eval |
| `sdlc-hooks` | allow / ask / block, build guardrails vs release gates, and why an approval prompt doesn't belong mid-build |
| `sdlc-parallel-work` | Worktrees and subagents — and the real ceiling on parallelism, which is review capacity, not machines |
| `sdlc-pr-review` | `REVIEW.md` passes, the Important-vs-nit line, the fix loop, findings feeding back into `CLAUDE.md` |
| `sdlc-close-loop` | Deterministic detection, σ-tiered response, headless runs, rehearsed rollback, per-environment autonomy |

Each skill carries the working templates — `intent.md`, `plan.md`, `CLAUDE.md`, `SKILL.md`, `REVIEW.md`, `settings.json` with a gate script, `bands.yaml`, an evals workflow — plus the leading and lagging indicators that say whether adopting it worked.

## Install

As a plugin:

```
/plugin marketplace add asaf-shitrit/ai-native-sdlc-skills
/plugin install ai-native-sdlc@ai-native-sdlc-skills
```

Or copy the skills straight in — personally, for every project:

```bash
git clone https://github.com/asaf-shitrit/ai-native-sdlc-skills
cp -r ai-native-sdlc-skills/plugins/ai-native-sdlc/skills/* ~/.claude/skills/
```

…or into one repo, so they ship with the code and get reviewed like it:

```bash
cp -r ai-native-sdlc-skills/plugins/ai-native-sdlc/skills/* .claude/skills/
```

## Where to start

Don't adopt these in stage order. Adopt by dependency — four of them have no prerequisites at all:

- **`sdlc-claude-md`** — cheapest, and it compounds into everything else
- **`sdlc-feedback-loop`** — the highest-leverage play here. Without it a human verifies every agent output, and that human is the bottleneck
- **`sdlc-plan`** — free; it's a habit, not infrastructure
- **`sdlc-hooks`** — for the one rule that must never break

Then work outward along the arrows in `ai-native-sdlc`.

## Scope

Written for solo developers and small teams. The enterprise-only material from the source playbook — MDM-deployed managed settings, sandbox admin keys, seat and spend administration, hosted scanning, chat-tool on-call — is deliberately left out, with links to the reference docs where the full detail matters.

## Credit

Distilled from Anthropic's [**The AI-Native SDLC playbook**](https://claude.com/blog/the-ai-native-sdlc-playbook) by Louis Claxton (August 2026), which is the canonical source and worth reading in full. The ideas, structure and example templates are theirs; this repo packages them as skills an agent loads at the moment they apply.

Not affiliated with or endorsed by Anthropic.

## License

MIT — see [LICENSE](LICENSE). Applies to the packaging in this repo.
