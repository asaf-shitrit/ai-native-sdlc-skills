---
name: sdlc-claude-md
description: Move institutional knowledge out of heads and wikis into files the agent reads — a short CLAUDE.md for repo context, and skills for rules that must apply consistently. Use when the agent repeats a mistake, when onboarding knowledge is undocumented or stale, when asked to write or trim a CLAUDE.md, when running /init and deciding what to keep, when a convention is enforced inconsistently, or when deciding whether something belongs in CLAUDE.md, a skill, or just the prompt. Covers the boundary between the three, keeping the file short, and why a skill alone can't guarantee anything.
---

# Knowledge as files

## Which container

| The knowledge is | Put it in |
|---|---|
| What a new joiner needs on day one for *this repo* — commands, conventions, layout, the traps | `CLAUDE.md` |
| A rule that must apply **consistently**, usually across repos, with someone who owns it | a **skill** |
| Relevant to this task only | the prompt |

The two failure modes are symmetric: a cross-cutting standard buried in one repo's `CLAUDE.md` where only that repo benefits, and a repo-specific quirk written up as a skill that then fires everywhere and gets ignored.

## CLAUDE.md

`/init` writes a first draft from what's actually in the repo. Treat it as raw material — generated files are always too long, and length is the thing that kills this file.

Cut to what someone needs on day one: **how to build, test and lint; the conventions that actually get enforced; the layout; and whatever the agent keeps getting wrong.** Delete the rest, including anything true of every repo in the language.

**Keep it under a page.** It's read at the start of every session, so a stale paragraph costs context forever and pays nothing.

Commit it at the repo root. One shared version, changes reviewed like code.

```markdown
# orders-service

## Commands
- Dev: bun run dev
- Test: bun test          (unit)
- Test: bun test:e2e      (needs `docker compose up -d`)
- Check: bun run check    (types + lint; CI runs this)

## Conventions
- Money in minor units as integers. Never floats, never Decimal strings.
- Times are UTC instants at rest; format only at the edge.
- Every route handler has a test that exercises the error path.
- Zod schemas live next to the route, not in a shared types file.

## Layout
- routes/ HTTP only — no business logic
- domain/ pure, no imports from routes/ or db/
- db/ queries and migrations; generated types in db/gen (do not edit)

## Known traps
- Don't upgrade deps to fix a type error; ask first.
- webhooks/stripe.ts must stay idempotent — it gets replayed.
- The seed script wipes the dev DB; never run it against a URL you didn't check.
```

**The rule that keeps it alive:** when the agent makes the same mistake twice, the correction goes in the file. Once is noise, twice is a pattern, and this is where patterns belong. Review feeds it too (`sdlc-pr-review`) — a finding raised for the second time should land here as part of that review, which closes the loop, since review reads this file.

Add a verification section as well (`sdlc-feedback-loop`).

## Skills

A skill is a folder with a `SKILL.md`: frontmatter describing **when it should fire**, body describing **what to do**.

Write one by starting from a rule that's enforced inconsistently today. Get it from whoever owns the rule rather than from folklore. Then:

**Test that it triggers.** Ask for the relevant task phrased three or four different ways and confirm it loads each time. A skill that doesn't fire isn't a weak control — it's no control, and it's worse than nothing because people believe it's there.

Ship it in the repo under `.claude/skills/` so it travels with the code, or distribute it as a plugin when it should apply everywhere. When the rule changes, change the skill; everyone picks it up next session.

```markdown
---
name: migration-safety
description: Rules for schema changes. Use whenever adding, editing or
  reviewing a migration, altering a table, or changing a column's type
  or nullability.
---

# Migration safety

Deploys are rolling, so old and new code run against the same schema
for a few minutes. Every migration must be safe for both.

- Additive first. Add a nullable column, backfill, then enforce —
  three deploys, not one.
- Never rename or drop in the same release that stops using the
  column. Drop it a release later.
- No table rewrites on tables over ~1M rows without a written plan;
  lock the table and the API stalls.
- Every migration has a tested down path, or an explicit comment
  saying why it is irreversible.

Before finishing, print the migration and state which of the rules
above applies to it.
```

## A skill does not guarantee anything

It makes compliance *likely* — it's read at the moment the work happens, which is the right moment. But nothing forces a session to follow it.

**So anything that must hold without exception needs something deterministic behind it**: a hook that blocks the action (`sdlc-hooks`), or a review pass that re-checks at the PR (`sdlc-pr-review`). Use the skill to make violations rare and the hook to make them impossible. Treating an advisory control as a guarantee is the most common mistake in this whole area.

## Worth watching

Repeat mistakes that the file should already have caught — if they persist, either the correction never landed or the file is too long to be read carefully. For skills: review findings citing a rule the skill covers should trend toward zero, and if they don't, the skill isn't firing or has drifted from the real rule.

---

*The practices here follow the AI-native SDLC described in Anthropic's [The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) (Louis Claxton, August 2026) — the canonical source, and worth reading in full. The wording and all examples in this file are original. Unofficial; not affiliated with or endorsed by Anthropic.*
