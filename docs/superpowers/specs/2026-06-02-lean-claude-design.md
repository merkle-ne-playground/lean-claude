# lean-claude — Design Spec

**Date:** 2026-06-02
**Repo:** `github.com/merkle-ne-playground/lean-claude` (public)
**Type:** Claude Code plugin + marketplace

## Purpose

A Claude Code plugin that helps users cut token/context spend without losing output quality. Bundles a guidelines skill (model-facing) plus two slash commands (per-user audit + setup) and human docs. Installable via the standard plugin marketplace flow.

## Why it exists

Token spend on Claude Code is dominated by avoidable patterns: subagents inheriting the most expensive model, giant SKILL.md bodies loaded on invocation, per-turn skill-listing bloat, cache-busting habits, and whole-file reads. These are fixable with config + working discipline. lean-claude packages the fixes so anyone can install them in one step instead of rediscovering them.

## Install path

```
/plugin marketplace add merkle-ne-playground/lean-claude
/plugin install lean-claude@lean-claude
```

Repo root doubles as the marketplace (single-plugin repo), mirroring the caveman plugin layout.

## Repo structure

```
lean-claude/
├── .claude-plugin/
│   ├── marketplace.json     # marketplace manifest listing the one plugin
│   └── plugin.json          # plugin manifest (name, version, description, author)
├── skills/
│   └── token-efficiency/
│       ├── SKILL.md         # thin core (~3K tokens): the practices + index
│       └── reference/       # deep-dive, Read on demand (progressive disclosure)
│           ├── subagent-routing.md
│           ├── cache-hygiene.md
│           └── big-skill-refactor.md
├── commands/
│   ├── token-audit.md       # /token-audit  — read-only, recommends
│   └── token-setup.md       # /token-setup  — ask-before-write
├── README.md
└── LICENSE                  # MIT
```

## Components

### 1. `token-efficiency` skill (model-facing)

Practices what it preaches — progressive disclosure. SKILL.md holds the rules tersely; depth lives in `reference/` loaded only when a topic is engaged.

Core rules covered:
- **Subagent delegation by model tier** — Haiku for read-only search/fan-out, Sonnet for edits/tests/refactors, Opus only for architecture/debugging/synthesis. Never Haiku for analysis (shallow). Subagents must not silently inherit the top model.
- **Prompt-cache hygiene** — caching is automatic; don't `/clear` mid-task, keep turns <5 min apart, append don't rewrite early context, stable tool order.
- **Narrow reads** — grep first, read the hit with offset/limit; don't re-read files after Edit.
- **`/clear` discipline** — clear between unrelated tasks; old context is paid every turn.
- **Big-skill signal** — SKILL.md bodies load on invocation; >20KB → split into thin core + on-demand reference files.
- **Skill-listing trim** — `skillOverrides` (`name-only`/`off`) shrinks the per-turn listing; `skillListingBudgetFraction` caps it.

Trigger phrases: "token efficient", "save tokens", "be brief", "reduce context", "lean".

`reference/` files:
- `subagent-routing.md` — full tier table, workflow per-agent assignment, examples.
- `cache-hygiene.md` — how Claude Code caching works, the 5-min TTL, do/don't list.
- `big-skill-refactor.md` — step-by-step splitting a fat SKILL.md into progressive disclosure.

### 2. `/token-audit` (prompt command, read-only)

Instructs Claude to:
1. Read the user's active `settings.json` (resolve `CLAUDE_CONFIG_DIR`, fallback `~/.claude`).
2. `wc -c` every SKILL.md in the skills dirs; flag any >20KB.
3. List enabled skills; recommend `skillOverrides` (`name-only` for by-name skills, `off` for unused) — recommend only, never write.
4. Check config for token wins: missing `skillOverrides`, large `skillListingMaxDescChars`, model default, etc.
5. Output a prioritized markdown report with concrete recommended changes the user can hand to `/token-setup`.

### 3. `/token-setup` (prompt command, ask-before-write)

Instructs Claude to apply recommended config, **confirming before each write**:
- Scaffold `skillOverrides` into `settings.json` (merge, never replace).
- Offer to drop a model-routing-policy memory.
- Point user to install caveman (link), not bundled.
Every mutation gated on explicit user confirmation.

### 4. README.md (human-facing)

The why + how, one-step install, a short practices summary, and a clearly-labelled link to the **caveman** plugin (third-party, JuliusBrussee, MIT) as a complementary install — referenced, not redistributed.

## Design decisions

- **No SessionStart hook.** Injecting guidelines every session burns tokens, defeating the plugin's purpose. The skill is on-demand only.
- **caveman referenced, not bundled.** Respect its license/authorship; link from README.
- **Commands are prompt-type markdown**, not node scripts — portable, zero runtime dependency.
- **Audit is read-only; setup is ask-before-write.** No silent mutation of user config.
- **Single-plugin marketplace repo** — repo root is the marketplace, mirroring caveman.

## Out of scope (YAGNI)

- No telemetry / token-usage dashboard (caveman-stats already covers session stats).
- No automatic model switching of the main thread (not programmatically possible; documented as a manual `/model` / `/fast` habit instead).
- No bundled third-party plugins.

## Success criteria

- `/plugin marketplace add merkle-ne-playground/lean-claude` then install works clean.
- `/token-audit` produces an actionable report against a real config without writing anything.
- `/token-setup` applies changes only after per-write confirmation, merging not replacing.
- `token-efficiency` skill loads thin and pulls reference files only on demand.
