# lean-claude Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `lean-claude` Claude Code plugin (token/context-efficiency guidelines skill + audit/setup commands + docs) and publish it as an installable marketplace at `github.com/merkle-ne-playground/lean-claude`.

**Architecture:** Single-plugin marketplace repo (root = marketplace, mirrors caveman layout). Manifests are JSON; the skill and commands are markdown. The skill uses progressive disclosure (thin SKILL.md + `reference/` loaded on demand). Commands are prompt-type markdown. No hooks. Verification is JSON-validity + structural checks + a real plugin install.

**Tech Stack:** Claude Code plugin manifests (marketplace.json, plugin.json), markdown skills/commands, git, gh CLI.

---

## File map

| File | Responsibility |
|---|---|
| `.claude-plugin/marketplace.json` | Marketplace manifest listing the one plugin |
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, desc, author) |
| `skills/token-efficiency/SKILL.md` | Thin core practices + index into reference/ |
| `skills/token-efficiency/reference/subagent-routing.md` | Model-tier routing detail |
| `skills/token-efficiency/reference/cache-hygiene.md` | Prompt-cache do/don't |
| `skills/token-efficiency/reference/big-skill-refactor.md` | Splitting a fat SKILL.md |
| `commands/token-audit.md` | `/token-audit` — read-only recommender |
| `commands/token-setup.md` | `/token-setup` — ask-before-write applier |
| `README.md` | Human docs + install + caveman link |
| `LICENSE` | MIT |

---

## Task 1: Plugin manifests

**Files:**
- Create: `.claude-plugin/marketplace.json`
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: Write `plugin.json`**

```json
{
  "name": "lean-claude",
  "description": "Token and context efficiency for Claude Code: a guidelines skill plus /token-audit and /token-setup commands. Cuts avoidable spend without losing output quality.",
  "version": "0.1.0",
  "author": { "name": "Augustin Gottlieb", "url": "https://github.com/merkle-ne-playground/lean-claude" }
}
```

- [ ] **Step 2: Write `marketplace.json`**

```json
{
  "name": "lean-claude",
  "owner": { "name": "Merkle NE Playground" },
  "plugins": [
    { "name": "lean-claude", "source": "./", "description": "Token/context efficiency guidelines, audit, and setup for Claude Code." }
  ]
}
```

- [ ] **Step 3: Verify both JSON files parse**

Run: `node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json')); JSON.parse(require('fs').readFileSync('.claude-plugin/marketplace.json')); console.log('ok')"`
Expected: `ok`

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin && git commit -m "feat: plugin + marketplace manifests"
```

---

## Task 2: token-efficiency skill (core)

**Files:**
- Create: `skills/token-efficiency/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Frontmatter + thin body. Keep under ~3K tokens. Exact frontmatter:

```markdown
---
name: token-efficiency
description: Cut token and context spend in Claude Code without losing quality. Use when the user says "token efficient", "save tokens", "be brief", "reduce context", "lean", or asks how to lower usage. Covers subagent model routing, prompt-cache hygiene, narrow reads, /clear discipline, big-skill refactor, and skill-listing trims.
---
```

Body sections (terse, fragment-friendly), each ending with a pointer to the reference file where relevant:
1. **Subagent routing** — one-line tier rule (Haiku=search, Sonnet=edits, Opus=architecture; never Haiku for analysis). "Detail: reference/subagent-routing.md".
2. **Cache hygiene** — caching auto; don't /clear mid-task; turns <5min; append don't rewrite; stable tool order. "Detail: reference/cache-hygiene.md".
3. **Narrow reads** — grep first; offset/limit; no re-read after Edit.
4. **/clear discipline** — clear between unrelated tasks.
5. **Big-skill signal** — SKILL.md body loads on invoke; >20KB → split. "Detail: reference/big-skill-refactor.md".
6. **Skill-listing trim** — skillOverrides name-only/off; skillListingBudgetFraction.

Rule: body names each reference file by relative path so the model knows to Read it on demand. Do NOT inline the reference content here.

- [ ] **Step 2: Verify frontmatter + size**

Run: `head -5 skills/token-efficiency/SKILL.md && wc -c skills/token-efficiency/SKILL.md`
Expected: valid `---` frontmatter with name/description; byte count < 4000.

- [ ] **Step 3: Commit**

```bash
git add skills/token-efficiency/SKILL.md && git commit -m "feat: token-efficiency skill core"
```

---

## Task 3: Skill reference files

**Files:**
- Create: `skills/token-efficiency/reference/subagent-routing.md`
- Create: `skills/token-efficiency/reference/cache-hygiene.md`
- Create: `skills/token-efficiency/reference/big-skill-refactor.md`

- [ ] **Step 1: Write `subagent-routing.md`**

Full tier table (Haiku/Sonnet/Opus → task types), the "never Haiku for analysis" rule with rationale, workflow per-agent assignment guidance, and 2 concrete dispatch examples (one search agent on Haiku, one builder on Sonnet). Mirror the content of the user's own model-routing-policy memory but written generically for any user.

- [ ] **Step 2: Write `cache-hygiene.md`**

How Claude Code prompt caching works (automatic, ~5-min TTL), why it matters, and a do/don't list: don't /clear mid-task, keep turns <5min apart, append don't rewrite early context, keep tool order stable, beware long idle gaps.

- [ ] **Step 3: Write `big-skill-refactor.md`**

Step-by-step: measure SKILL.md size, identify what's load-bearing vs on-demand, extract detail into reference/ files, leave a thin core that names them, verify the skill still triggers. Note the >20KB rule of thumb.

- [ ] **Step 4: Verify all three exist and are non-empty**

Run: `for f in subagent-routing cache-hygiene big-skill-refactor; do wc -l skills/token-efficiency/reference/$f.md; done`
Expected: three files, each > 10 lines.

- [ ] **Step 5: Commit**

```bash
git add skills/token-efficiency/reference && git commit -m "feat: skill reference detail files"
```

---

## Task 4: /token-audit command

**Files:**
- Create: `commands/token-audit.md`

- [ ] **Step 1: Write the command**

Prompt-type markdown. Frontmatter `description: Audit this machine's Claude Code config and skills for token-saving opportunities (read-only).` Body instructs Claude to, READ-ONLY (never write):
1. Resolve config dir: `$CLAUDE_CONFIG_DIR` else `~/.claude`. Read its `settings.json`.
2. `wc -c` every `SKILL.md` under the skills dirs; flag >20KB.
3. List enabled skills; recommend `skillOverrides` — `name-only` for by-name/client skills, `off` for unused. Recommend only.
4. Check config: missing `skillOverrides`, oversized `skillListingMaxDescChars`, main `model` default.
5. Output a prioritized markdown report; end by suggesting `/token-setup` to apply.
State explicitly in the command body: this command must not modify any file.

- [ ] **Step 2: Verify frontmatter**

Run: `head -3 commands/token-audit.md`
Expected: `---` frontmatter with a `description:` line.

- [ ] **Step 3: Commit**

```bash
git add commands/token-audit.md && git commit -m "feat: /token-audit command"
```

---

## Task 5: /token-setup command

**Files:**
- Create: `commands/token-setup.md`

- [ ] **Step 1: Write the command**

Prompt-type markdown. Frontmatter `description: Apply recommended token-saving config to this machine, confirming before each write.` Body instructs Claude to apply changes, CONFIRMING BEFORE EACH WRITE:
1. Read settings.json first (merge, never replace arrays/objects).
2. Offer to add a `skillOverrides` scaffold (from `/token-audit` output if present).
3. Offer to drop a model-routing-policy memory file.
4. Point the user to install caveman: `https://github.com/JuliusBrussee/caveman` (referenced, not bundled).
Each mutation gated on an explicit yes. State explicitly: never write without confirmation.

- [ ] **Step 2: Verify frontmatter**

Run: `head -3 commands/token-setup.md`
Expected: `---` frontmatter with a `description:` line.

- [ ] **Step 3: Commit**

```bash
git add commands/token-setup.md && git commit -m "feat: /token-setup command"
```

---

## Task 6: README + LICENSE

**Files:**
- Create: `README.md`
- Create: `LICENSE`

- [ ] **Step 1: Write LICENSE**

Standard MIT license text, copyright `2026 Augustin Gottlieb / Merkle`.

- [ ] **Step 2: Write README.md**

Sections: title + one-line pitch; Install (the two `/plugin` commands from the spec); What's inside (skill + two commands, one line each); Practices summary (the six core rules, terse); Complementary: link to **caveman** (third-party, JuliusBrussee, MIT) — referenced not bundled; License.

- [ ] **Step 3: Verify install commands present**

Run: `grep -c "plugin marketplace add merkle-ne-playground/lean-claude" README.md`
Expected: `1` or more.

- [ ] **Step 4: Commit**

```bash
git add README.md LICENSE && git commit -m "docs: README and MIT license"
```

---

## Task 7: Publish + verify install

**Files:** none (remote)

- [ ] **Step 1: Create the GitHub repo and push**

Run: `cd ~/projects/lean-claude && gh repo create merkle-ne-playground/lean-claude --public --source=. --remote=origin --push`
Expected: repo created, branch pushed.

- [ ] **Step 2: Verify marketplace is addable**

Run: `gh api repos/merkle-ne-playground/lean-claude/contents/.claude-plugin/marketplace.json --jq '.name' | head` (after push)
Expected: prints `marketplace.json` blob metadata (path resolves) — confirms manifest is at the expected path on the remote.

- [ ] **Step 3: Manual install smoke test (user-run)**

In a Claude Code session: `/plugin marketplace add merkle-ne-playground/lean-claude` then `/plugin install lean-claude@lean-claude`. Confirm the `token-efficiency` skill and `/token-audit` + `/token-setup` commands appear. (This step is user-run; note it in handoff.)

---

## Notes for the implementer

- This repo IS a token-efficiency demonstration. Keep every file lean; the skill must practice progressive disclosure (thin core, on-demand reference).
- No SessionStart hook — deliberate (would burn tokens every session).
- Commands and skill are markdown only — no node runtime dependency.
- caveman is referenced by URL only; do not copy its files in.
