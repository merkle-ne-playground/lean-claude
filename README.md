# lean-claude

Token and context efficiency for Claude Code — spend fewer tokens without losing quality.

## Install

```
/plugin marketplace add merkle-ne-playground/lean-claude
/plugin install lean-claude@lean-claude
```

## What's inside

- **`token-efficiency` skill** — loads the six core efficiency rules into Claude's working context when you ask it to "save tokens", "be lean", or "reduce context spend".
- **`/token-audit`** — read-only inspection of your Claude Code config and installed skills; flags oversized SKILL.md files and missing token-saving config, and outputs a prioritised list of recommended changes.
- **`/token-setup`** — interactive wizard that applies the recommended changes from `/token-audit`, with an explicit confirmation prompt before every write.

## Practices

Six rules that the `token-efficiency` skill enforces:

1. **Subagent routing** — pin each agent to the right model tier: Haiku for read-only/search tasks, Sonnet for edits and generation, Opus for architecture and deep reasoning.
2. **Cache hygiene** — don't `/clear` mid-task, keep turns under 5 minutes apart, append context rather than rewriting early turns, and keep tool-call order stable.
3. **Narrow reads** — grep before reading; read only the hit range with `offset`/`limit`; never re-read a file after an Edit.
4. **`/clear` discipline** — clear only between unrelated tasks; stale task context is re-billed on every turn.
5. **Big-skill refactor** — keep SKILL.md bodies under 20 KB; move detail into reference files that are read on demand, not on every invocation.
6. **Skill-listing trim** — use `skillOverrides` to set unused skills to `name-only` or `off`, and cap `skillListingMaxDescChars` to shrink the per-turn listing cost.

## Complementary

[**caveman**](https://github.com/JuliusBrussee/caveman) by JuliusBrussee (MIT) — commit-message discipline and additional context hygiene. Third-party plugin; referenced here for discoverability, not bundled.

## License

MIT — see [LICENSE](LICENSE).
