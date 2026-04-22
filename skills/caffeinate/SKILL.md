---
name: caffeinate
description: Keep the Claude Code prompt cache warm while the user is away so the session doesn't eat a full cache rewrite on return. Use when the user says "caffeinate", "keep alive", "stay warm", "don't let the cache die", or is stepping away mid-session and wants to resume cheaply.
user_invocable: true
---

# Caffeinate

Anthropic's prompt cache has a **5-minute TTL**. Any idle gap over 5 minutes expires the cached context — the next turn pays `cache_creation` (write rate) instead of `cache_read` (read rate). For long sessions with a loaded context, that's a 20–60% cost bump every time the user walks away.

This skill pings the session just under that window so the cache stays warm.

## The move

`/caffeinate [count]` — hand off to `/loop` on a 4-minute cadence with a trivial keep-alive turn, and stop after `count` ticks.

- **Default count: 15** (~1 hour of keep-alive)
- **Reliable range: up to 30** (~2 hours). Beyond that, the self-count drifts — see below.
- **Max: 60** (~4 hours — ask the user to confirm; offer the cron fallback for long runs).
- `count` is the stop condition. Track tick number in your turn state; when the loop fires for tick `N+1`, don't ping — instead, cancel the loop and tell the user caffeinate finished.

### Counting caveat

`/loop` has no tick budget of its own — every firing is a separate prompt invocation, so the model has to count its own prior acknowledgments in conversation history to know which tick it's on. Reliable up to ~30. For higher counts with lots of intervening messages, the model can miscount and stop early or overshoot. For long bulletproof runs, prefer `/schedule` with N one-shot triggers at 4/8/…/N×4 min.

### Pre-flight check

Before starting, make sure no other `/loop` is already running in this session — caffeinate + an existing loop = double-ping (wasted turns, noisy log). If one is running, either cancel it first or skip caffeinate entirely (the existing loop already keeps the cache warm).

4 minutes (240s) sits safely under the 300s TTL with ~60s of slack for scheduling jitter. Each loop tick re-reads the conversation, which re-references the cached prefix → TTL resets.

### Invocation

```
/caffeinate          # 15 ticks (~60 min)
/caffeinate 10       # 10 ticks (~40 min)
/caffeinate 30       # 30 ticks (~2 hours)
```

On start, tell the user the total planned duration so they can course-correct (e.g. "caffeinating for ~60 min / 15 ticks").

### Stop conditions (any of)

1. Tick counter reaches `count` — say "caffeinate done, cache warm until ~HH:MM" and stop.
2. User interrupts or says "stop caffeinating" — cancel loop immediately.
3. User sends any other prompt — the natural turn warms the cache on its own; cancel the loop to avoid double-pinging.

## When to reach for it

- User says "I'm stepping away for an hour, keep this warm"
- Mid-debug session with a loaded context the user wants to resume cheaply
- Long-running background work where the user will check back intermittently

## When NOT to use it

- **Short breaks** (under 5 min) — cache stays warm on its own
- **Session you're done with** — just let it expire; paying to keep a dead cache alive is worse than paying one rewrite
- **Active work** — if turns are already happening every few minutes, caffeinate is redundant
- **Cost-sensitive contexts where the cached prefix is small** — the rewrite cost isn't worth the keep-alive cost

Rough rule: caffeinate pays off when **cached-tokens × rewrite-premium > tick-cost × ticks-until-resume**. Big context + long break = yes. Small context or short break = no.

## Notes

- Cache TTL regression traced to early March 2026 ([claude-code#46829](https://github.com/anthropics/claude-code/issues/46829)). Anthropic still offers a 1-hour cache variant at extra cost; caffeinate is the DIY alternative on the default 5-min tier.
- Don't confuse with macOS `caffeinate(1)` — that keeps the *machine* awake; this keeps the *cache* alive.
- Each tick still costs a turn. Prefer a realistic `count` over "just in case" padding.
