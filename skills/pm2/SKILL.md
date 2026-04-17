---
name: pm2
description: PM2 process management — list, logs, restart, stop, and related ops. Use when the user asks to check, restart, stop, or read logs for a PM2-managed process, either locally or over SSH.
user_invocable: true
---

# PM2

Process management actions for PM2. Run locally by default; if the user references a remote host ("prod", "the droplet") and there's an SSH alias configured, wrap the command in `ssh <alias> "..."`.

## List

```bash
pm2 list
```

Report process states. **Flag at the top** anything in `errored` / `stopped`, high restart counts (crash loops), or memory climbing toward a `max_memory_restart` threshold.

## Logs

```bash
pm2 logs <name> --lines 100 --nostream
```

`--nostream` prints and exits (no follow). 100 lines is usually enough to spot recent failures. Surface errors/exceptions at the top of your summary — don't bury them in raw output.

**Follow mode** (live tail) — run in a background shell:
```bash
pm2 logs <name> --lines 50
```

Log files also live at `~/.pm2/logs/` for raw access.

## Restart

```bash
pm2 restart <name>
```

If env changed (e.g. after pulling secrets), use `--update-env`:
```bash
pm2 restart <name> --update-env
```

**`restart all`** cycles every process — confirm before running.

After restart: show `pm2 list` so the user sees the new uptime. If the process flips to `errored`, immediately pull logs.

`pm2 reload` (zero-downtime) only works for cluster-mode processes. Default to `restart`.

## Stop

```bash
pm2 stop <name>
```

Keeps the process in PM2's list — can be restarted later. **`stop all`** — confirm first.

## Delete (destructive)

```bash
pm2 delete <name>
```

Removes from PM2 entirely. Don't alias "stop" to this — do the explicit action. Confirm before running.

## Flush logs (disk cleanup)

```bash
pm2 flush
```

Truncates all log files. Useful when logs have grown large. Non-destructive to processes.

## Remote execution

If the user means a remote host, wrap the command. Example with `prod` alias:

```bash
ssh prod "pm2 list"
ssh prod "pm2 logs polaris-web --lines 100 --nostream"
ssh prod "pm2 restart polaris-web"
```

## When no process name is given

Run `pm2 list` first, show the user what's available, then ask which process.
