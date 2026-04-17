---
name: pm2-stop
description: Stop a PM2 process without removing it from the process list. Use when the user says "stop X", "pm2 stop X", "take X down", or wants to temporarily halt a service.
user_invocable: true
---

# PM2 Stop

Stop a PM2-managed process (keeps it in the list — can be restarted later).

## Usage

The argument is the process name or id (e.g. `/pm2-stop polaris-web`).

## Single process

```bash
pm2 stop $NAME
```

## All

```bash
pm2 stop all
```

Confirm before running — stops every managed service.

## Remote (via SSH)

```bash
ssh <alias> "pm2 stop $NAME"
```

## After running

Run `pm2 list` to confirm the `stopped` state.

## Notes

- `pm2 stop` keeps the process in PM2's list — use `/pm2-restart $NAME` to bring it back.
- `pm2 delete $NAME` **removes** the process from PM2 entirely (different command, destructive — not handled by this skill). If the user says "delete" or "remove", don't alias to stop — do the explicit thing and confirm.
- If the process keeps restarting after `pm2 stop`, PM2's watch mode or a crash loop may be misconfigured — check `pm2 describe $NAME`.
