---
name: pm2-restart
description: Restart a PM2 process. Use when the user says "restart X", "pm2 restart X", "bounce X", or when a service is unhealthy and needs a cycle.
user_invocable: true
---

# PM2 Restart

Restart a PM2-managed process.

## Usage

The argument is the process name or id (e.g. `/pm2-restart polaris-web`). Without an argument, restart everything — confirm with the user first, since that's heavy.

## Single process

```bash
pm2 restart $NAME
```

## All processes

```bash
pm2 restart all
```

Confirm before running — this cycles every managed service.

## With env update

If the user has just changed environment variables (e.g. after pulling secrets), use `--update-env` so the new env is picked up:

```bash
pm2 restart $NAME --update-env
```

Plain `pm2 restart` keeps the old env.

## Remote (via SSH)

```bash
ssh <alias> "pm2 restart $NAME"
```

## After running

1. Report the exit status.
2. Run `pm2 list` to show the new state (uptime should reset to seconds).
3. If the user expects logs to follow, offer `/pm2-logs $NAME`.

## Notes

- `pm2 reload` (zero-downtime, cluster-mode only) is different from `pm2 restart` (kill + respawn). Use `reload` only if the process was started in cluster mode.
- If a restart puts the process into `errored` state, the startup is failing — pull logs immediately.
