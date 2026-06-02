# Cache Hygiene — Full Detail

## How Claude Code prompt caching works

Claude Code uses automatic prompt caching. When a conversation turn shares a long common prefix with a recent turn (same system prompt, same tool list, same prior messages), the API returns a **cache hit** and charges input tokens at a fraction of the normal rate (roughly 10% for Haiku/Sonnet, 30% for Opus).

The TTL is approximately **5 minutes**. If more than 5 minutes elapse between turns, the cache entry expires and the next turn pays full input price again.

Cache hits are invisible in the UI but appear in API usage metadata (`cache_read_input_tokens`). On a long session, cache hits can reduce input costs by 60–80%.

## Why cache hits matter

Every input token in the conversation window is charged on every turn — system prompt, all prior messages, all tool definitions. A 50-turn session on a 10K-token context costs 500K input tokens without caching. With a warm cache, the first turn is full price and subsequent turns are ~10% price, collapsing that to ~55K effective tokens.

## Do / Don't list

**DO:**
- Keep turns under 5 minutes apart on active tasks.
- Let the session run continuously; don't context-switch away mid-task.
- Append new information rather than rewriting earlier messages.
- Keep your tool list stable across turns (same tools, same order).
- Use `/clear` deliberately — only between truly unrelated tasks.

**DON'T:**
- Run `/clear` mid-task. It destroys the shared prefix, voiding the cache.
- Leave long idle gaps (lunch, meeting) mid-session without expecting a cold restart.
- Reorder tools or rewrite system context between turns — breaks cache matching.
- Spawn subagents with different system prompts when they could share the parent prefix.

## Idle gap recovery

If you return after a long break, the cache is cold. Accept the first turn is full price, then keep the next turns tight to rebuild a warm cache. Don't force a `/clear` "to start fresh" — that extends the cold window.
