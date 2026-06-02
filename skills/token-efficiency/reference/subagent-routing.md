# Subagent Routing — Full Detail

## Model-tier table

| Tier   | Task types                                                         | Notes                                      |
|--------|--------------------------------------------------------------------|--------------------------------------------|
| Haiku  | Read-only search, grep/fan-out, file listing, counting, triage    | Fast + cheap; no depth for analysis        |
| Sonnet | Edits, test writing, refactors, code generation, file transforms   | Balanced; default workhorse                |
| Opus   | Architecture decisions, complex multi-file debugging, synthesis    | Slow + expensive; reserve for hard calls   |

## The "never Haiku for analysis" rule

Haiku will produce analysis-shaped output — headers, bullet points, a conclusion — but the reasoning is shallow. In code review, security audit, or test-critique tasks it will miss subtle issues while appearing thorough. The cost saving is real; the quality loss is invisible until it matters. Use Sonnet minimum for anything that requires judgment.

## Assigning models in multi-agent workflows

Dispatch pattern:

1. **Haiku finders** — fan-out to N parallel agents, each doing a single grep/read/list over a file region. Collect results.
2. **Sonnet transformers** — one agent per file/chunk that needs editing. Receives only the relevant hit from the finder.
3. **Opus reasoner** (optional) — single orchestrator that receives the transformed outputs and produces the synthesis or final decision.

This keeps cheap tokens on cheap work and reserves expensive tokens for where they add value.

## Concrete dispatch examples

### Example A — read-only search agent (Haiku)

```json
{
  "model": "claude-haiku-4-5",
  "task": "Find all usages of `fetchUser` in src/. Return file paths and line numbers only.",
  "tools": ["Bash(grep)", "Read"]
}
```

Haiku reads, never writes. Output is a list of hits passed upstream.

### Example B — file-editing builder (Sonnet)

```json
{
  "model": "claude-sonnet-4-5",
  "task": "Refactor fetchUser calls in src/api/users.ts to use the new UserService.get() interface.",
  "context": "<hit list from finder>",
  "tools": ["Read", "Edit"]
}
```

Sonnet receives only the relevant file + hit context, not the whole repo.

## Main-thread model note

The main conversation thread cannot be auto-switched programmatically. The user must run `/model` or `/fast` to change it. Suggestion: on long mechanical sessions (bulk renaming, mass formatting, repetitive search), prompt the user to switch to `/fast` (Haiku) mode at the start. Switch back before any analysis step.
