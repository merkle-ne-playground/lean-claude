# Big-Skill Refactor — Step-by-Step Guide

## Why size matters

SKILL.md body is loaded into context **on every invocation** of the skill — not on demand, not lazily. A 30 KB SKILL.md costs 30 KB of input tokens every single time the skill is triggered, for every turn of the session. At scale (many invocations, many turns), this is the single largest avoidable context cost in a skill-heavy setup.

Rule of thumb: **>20 KB → split into thin core + reference files.**

Reference files are read only when the model explicitly calls `Read` on them. The SKILL.md just names them; they cost nothing until needed.

## Step-by-step refactor

### 1. Measure current size

```bash
wc -c skills/my-skill/SKILL.md
```

If output is over 20480 (20 KB), proceed.

### 2. Identify load-bearing rules vs. on-demand detail

Read through the skill body and categorize each section:

- **Load-bearing** — triggers, invocation conditions, short rules the model must apply immediately, pointers to reference files. Keep in SKILL.md.
- **On-demand detail** — examples, rationale, edge cases, step-by-step guides, tables. Move to reference files.

A rule is load-bearing if the model would make a wrong decision without it on the first invocation. It is on-demand if the model only needs it when going deeper.

### 3. Extract detail into reference/ files

Create a `reference/` subdirectory next to SKILL.md. One file per topic:

```
skills/my-skill/
  SKILL.md              ← thin core, <4 KB ideal
  reference/
    topic-a.md
    topic-b.md
    topic-c.md
```

Move the detail prose into those files. Keep each reference file focused on one topic.

### 4. Leave a thin core that names the reference files

In SKILL.md, replace the extracted detail with a one-line pointer:

```
Detail: reference/topic-a.md
```

The model will call `Read skills/my-skill/reference/topic-a.md` when it needs the depth. The path must be relative to the skill root so it resolves correctly.

### 5. Verify the skill still triggers

Run the skill. Confirm:
- The thin core loads on invocation (check context size).
- The trigger conditions still match correctly (frontmatter description unchanged).
- When deeper detail is needed, the model reads the reference file and applies it correctly.

Re-measure:

```bash
wc -c skills/my-skill/SKILL.md
```

Target: under 4000 bytes for frequently-triggered skills.
