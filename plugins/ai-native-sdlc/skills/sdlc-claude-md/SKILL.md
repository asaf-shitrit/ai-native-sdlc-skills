---
name: sdlc-claude-md
description: Turn institutional knowledge into files the agent reads — a tight CLAUDE.md for repo context and skills for policy that must be applied consistently. Use when the agent repeats a mistake, when onboarding context lives in people's heads or a stale wiki, when asked to write or trim a CLAUDE.md, when running /init and deciding what to keep, when a convention or standard is enforced inconsistently, and when deciding whether something belongs in CLAUDE.md, a skill, or just the prompt. Covers the CLAUDE.md template, the one-page rule, the twice-wrong rule, and the skill-vs-CLAUDE.md boundary.
---

# Institutional knowledge as files

Knowledge that used to sit in heads and on wikis becomes files the agent reads, maintained by the whole team, reviewed like code.

## Where does this belong?

| It is… | Put it in |
|---|---|
| Context a new joiner needs on day one for *this repo* — commands, conventions, architecture, the mistakes the team keeps making | `CLAUDE.md` |
| Knowledge that must be applied **consistently**, often across repos, with a named policy owner — a security standard, an API convention, a brand rule | a **skill** |
| True only for this one task | the prompt |

Do not write a skill for something that belongs in `CLAUDE.md` or a prompt. Do not put a cross-cutting policy in `CLAUDE.md` where only one repo sees it.

## CLAUDE.md

### Building it

1. **Run `/init`.** Claude generates a starting file from what it finds.
2. **Cut it down to what a new joiner needs on day one.** Generated files are always too long. Keep the build/test/lint commands, the conventions that actually matter, and the things the agent keeps getting wrong. Delete the rest.
3. **Check it into git at the repo root** so the team shares one version and changes get reviewed.
4. **Keep it under a page.** It is read at the start of every session; anything stale is burning context for no benefit.

### The twice-wrong rule

When Claude makes the same mistake twice, the correction goes into `CLAUDE.md`. Not the first time — once is noise. Twice is a pattern, and the file is where patterns go. PR review feeds this too (`sdlc-pr-review`): when a review flags the same mistake a second time, the correction lands in `CLAUDE.md` as part of that review, and because review reads `CLAUDE.md`, it is caught from the next PR onward.

### Template

```markdown
# Payments service

## Commands
- Build: make build
- Test: make test (unit), make itest (integration, needs docker)
- Lint: make lint (runs in CI; fix before pushing)

## Conventions
- Java 21, Spring Boot 3. No new Lombok.
- Money is always BigDecimal, never double.
- Every endpoint needs an integration test in src/itest.

## Architecture
- api/ holds REST controllers, core/ holds domain logic,
  adapters/ talks to external systems.
- Kafka events are defined in schemas/; never edit generated classes.

## Things Claude gets wrong
- Do not bump dependency versions; the platform team owns them.
- The legacy v1/ package is frozen; changes go in v2/.
```

Add a **Verifying your work** section too — see `sdlc-feedback-loop`.

## Skills

### Writing one

1. **Pick one piece of knowledge that is enforced inconsistently today.**
2. **Write it as a folder containing `SKILL.md`** — frontmatter says *when it triggers*, body says *what to do*. Write it from the policy owner's source of truth.
3. **Place it**: `.claude/skills/<name>/` in the repo so it ships with the code, or distribute organization-wide as a plugin.
4. **Test that it triggers.** Ask for the relevant task phrased several different ways and confirm the skill loads each time. An untriggered skill is not a control.
5. **When the policy changes, change the skill** and have the policy owner sign off. Everyone picks up the new version in their next session.

### Example

```markdown
---
name: secure-api-review
description: Apply the API security standard. Use whenever creating or
  modifying an external-facing endpoint, reviewing API code, or
  generating an OpenAPI spec.
---

# Secure API review

When you create or change an API endpoint:
1. Authentication: every endpoint requires the gateway JWT;
   no anonymous routes outside /health.
2. Input validation: validate request bodies against the OpenAPI
   schema and reject unknown fields.
3. Audit: every state-changing endpoint emits an audit event with
   actor, action, entity and timestamp.
4. Data classification: fields tagged pii in the schema must never
   appear in logs or error messages.

Run scripts/check-endpoints.sh and include its output in your summary.
```

### A skill is an advisory control

It makes the agent *likely* to apply the policy while the code is written. Nothing forces a session to comply. **A policy that must always hold needs something deterministic behind it** — a hook that blocks the action (`sdlc-hooks`) or a review pass that re-checks at the PR (`sdlc-pr-review`). The skill makes violations rare; the hook makes them close to impossible.

## Measuring it

- **Leading** — how often the agent repeats a mistake `CLAUDE.md` should have caught; time from a policy change being approved to the updated skill merging.
- **Lagging** — time to first merged PR for a new team member; PR review findings citing a policy, which should fall toward zero once the skill applies it at authoring time. If they do not fall, either the skill is not triggering or its text has drifted from the real policy.

---

*Distilled from Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) by Louis Claxton (August 2026), which is the canonical source. This is an unofficial repackaging into skill form; not affiliated with or endorsed by Anthropic.*
