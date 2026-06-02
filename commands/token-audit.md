---
description: Audit this machine's Claude Code config and skills for token-saving opportunities (read-only).
---

This command must not modify any file. All steps below are read-only inspection only.

## Steps

### 1. Resolve the config directory

- If `$CLAUDE_CONFIG_DIR` is set, use that path. Otherwise use `~/.claude`.
- Read `settings.json` from that directory. If it does not exist, note that no settings file was found and continue.

### 2. Measure every SKILL.md on this machine

Run `wc -c` on every file named `SKILL.md` found under the skills directories (check `~/.claude/skills/`, `$CLAUDE_CONFIG_DIR/skills/`, and any `skills/` paths referenced in `settings.json`).

Flag any SKILL.md whose byte count exceeds 20 480 bytes (20 KB) as a **refactor candidate**. For each candidate, note its path and size.

### 3. Analyse enabled skills and recommend overrides

List all skills visible in the config (from `settings.json` `skills` or `skillOverrides` keys, or by directory listing).

For each skill, classify it:
- **name-only** — skills that are invoked by explicit name or are client/project-specific (they don't need full descriptions loaded on every turn).
- **off** — skills that appear unused (no recent invocation evidence, no obvious match to current projects).

Produce a recommended `skillOverrides` block showing which skills to set to `"name-only"` and which to set to `"off"`. Do not write this — recommend only.

### 4. Check config for token wins

Inspect `settings.json` for the following issues and note each finding:

- **Missing `skillOverrides`** — if the key is absent entirely, every skill description is loaded on every turn.
- **`skillListingMaxDescChars` too large or absent** — a high value or missing cap inflates the per-turn skill listing. Recommend a value of 120–200 characters.
- **Default model** — if `model` is not set or is set to Opus/Sonnet for all tasks, flag that subagent routing (Haiku for read-only, Sonnet for edits, Opus for architecture) could reduce cost significantly.

### 5. Output a prioritised report

Produce a markdown report with the following structure:

```
# Token Audit Report

## Config path
<resolved path>

## Findings (highest impact first)

### 1. <finding title>
- Impact: <high/medium/low>
- Current: <what was found>
- Recommended: <what to change>

### 2. ...

## Oversized SKILL.md files (>20 KB)
| Path | Size |
|------|------|
| ...  | ...  |

## Recommended skillOverrides
```json
{ "skillOverrides": { ... } }
```

## Next step
Run `/token-setup` to apply any of the above changes interactively, with a confirmation prompt before each write.
```

End the report with the suggestion: **Run `/token-setup` to apply these recommendations.**
