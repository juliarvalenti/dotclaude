---
name: pm2-logs
description: Tail logs from a PM2-managed process. Use when the user says "pm2 logs X", "show logs for X", "what is X printing", or is debugging a PM2 service.
user_invocable: true
---

# PM2 Logs

Show recent log output from a PM2 process.

## Usage

The argument is the process name or PM2 id (e.g. `/pm2-logs polaris-web`). If no argument, run `pm2 list` first to see what's available.

## Default

```bash
pm2 logs $NAME --lines 100 --nostream
```

`--nostream` prints and exits (no follow) — 100 lines is usually enough to spot a recent failure.

## Follow mode

If the user wants live updates ("follow", "-f", "tail it"):

```bash
pm2 logs $NAME --lines 50
```

Run in a background shell so the conversation continues — user can stop it when ready.

## Remote (via SSH)

If the user references a remote host:

```bash
ssh <alias> "pm2 logs $NAME --lines 100 --nostream"
```

## After running

- If there are recent errors/exceptions, surface them at the top — don't bury in raw output.
- Note which stream (`out` vs `err`) the problem came from.
- If the log is empty, the process might not have logged yet, or may have been restarted and flushed. Check `pm2 list` for uptime.

## Notes

- `pm2 logs` combines stdout + stderr by default. Use `--err` or `--out` to filter.
- Log files live at `~/.pm2/logs/` — raw access if PM2's pager is mangling the output.
