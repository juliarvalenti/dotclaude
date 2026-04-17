---
name: pm2-list
description: Show PM2-managed processes with status, uptime, memory, and restart count. Use when the user says "pm2 list", "pm2 ls", "what's running in pm2", or needs to see process state.
user_invocable: true
---

# PM2 List

List processes managed by PM2.

## Local

```bash
pm2 list
```

## Remote (via SSH)

If the user references a remote host (e.g. "on prod", "on the droplet") and an SSH alias exists, run over SSH:

```bash
ssh <alias> "pm2 list"
```

## After running

- If no processes: say so.
- **Flag concerning states**:
  - `errored` / `stopped` — surface at the top
  - High restart count (`↺` column) — indicates a crash loop
  - Memory climbing toward a `max_memory_restart` threshold
  - Uptime shorter than expected given recent deploys

- If the user asks about a specific process by name, follow up naturally with `/pm2-logs <name>`.
