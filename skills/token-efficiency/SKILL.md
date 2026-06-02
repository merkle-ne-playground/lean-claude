---
name: token-efficiency
description: Cut token and context spend in Claude Code without losing quality. Use when the user says "token efficient", "save tokens", "be brief", "reduce context", "lean", or asks how to lower usage. Covers subagent model routing, prompt-cache hygiene, narrow reads, /clear discipline, big-skill refactor, and skill-listing trims.
---

## 1. Subagent routing

Route by task type, not convenience:
- **Haiku** — read-only search, fan-out grep, listing, counting. Never analysis.
- **Sonnet** — edits, tests, refactors, generation with context.
- **Opus** — architecture decisions, complex debugging, multi-file synthesis.

Never let subagents silently inherit the top-thread model. Pin each agent explicitly.

Detail: reference/subagent-routing.md

---

## 2. Cache hygiene

Prompt caching is automatic; your job is not to break it:
- Don't `/clear` mid-task — it voids the cache.
- Keep turns <5 min apart; idle gaps expire TTL.
- Append context; don't rewrite early turns.
- Keep tool-call order stable across turns.

Detail: reference/cache-hygiene.md

---

## 3. Narrow reads

Grep before reading. Read only the hit range with `offset`/`limit`. Never re-read a file after an Edit — the harness tracks state.

---

## 4. /clear discipline

Clear only between unrelated tasks. Every token in context is re-billed on every turn. Old task context = dead weight.

---

## 5. Big-skill signal

SKILL.md body loads into context on every invocation. Keep it thin.
- Rule of thumb: >20 KB → split into thin core + reference files.
- Reference files are read on demand, not on invocation.

Detail: reference/big-skill-refactor.md

---

## 6. Skill-listing trim

Reduce the per-turn skill-listing cost:
- `skillOverrides`: set unused skills to `name-only` or `off`.
- `skillListingBudgetFraction`: cap total listing size.
