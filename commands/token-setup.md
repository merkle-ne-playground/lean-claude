---
description: Apply recommended token-saving config to this machine, confirming before each write.
---

Never write without explicit user confirmation. Every mutation below must be gated on a clear "yes" from the user before proceeding. If the user says anything other than an affirmative, skip that step.

## Steps

### 1. Read current settings

Resolve the config directory: use `$CLAUDE_CONFIG_DIR` if set, else `~/.claude`. Read `settings.json` from that directory.

When applying any edit later, **merge** into the existing JSON — never replace or overwrite existing arrays or objects wholesale. Only add or update the specific keys being changed.

### 2. Offer to add a `skillOverrides` scaffold

If the user has already run `/token-audit`, use the recommended `skillOverrides` block from that output. Otherwise, list the skills currently visible in the config and ask the user which they want to set to `"name-only"` (invoked by name / client-specific) or `"off"` (unused).

Present the proposed `skillOverrides` block to the user. Ask: **"Apply this skillOverrides scaffold to settings.json? (yes/no)"**

Only if the user confirms: merge the `skillOverrides` key into `settings.json`, preserving all other existing keys.

### 3. Offer to add a model-routing-policy memory file

Offer to write a short memory file (e.g. `~/.claude/memory/model-routing-policy.md`) that describes the three-tier routing policy:

- **Haiku** — read-only tasks: search, grep, list, count, summarise. Never for analysis or generation with heavy context.
- **Sonnet** — edits, tests, refactors, code generation, most interactive tasks.
- **Opus** — architecture decisions, complex multi-file debugging, synthesis tasks requiring deep reasoning.

Present the proposed file path and content. Ask: **"Write model-routing-policy.md? (yes/no)"**

Only if the user confirms: write the file. Do not overwrite an existing file at that path without showing the diff and getting a second confirmation.

### 4. Point to the complementary caveman plugin

Recommend the **caveman** plugin by JuliusBrussee for commit-message discipline and further context hygiene:

> Install it separately: https://github.com/JuliusBrussee/caveman
>
> It is a third-party plugin — referenced here for discoverability, not bundled with lean-claude.

No write action required for this step.

### 5. Summary

After all confirmations are resolved (yes or no), print a brief summary:

- Which changes were applied.
- Which were skipped.
- Remind the user to run `/clear` before the next unrelated task to flush stale context.
