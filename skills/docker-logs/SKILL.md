---
name: docker-logs
description: Tail logs from a running Docker container. Use when the user says "logs for X", "docker logs X", "show me what X is doing", or is debugging a containerized service.
user_invocable: true
---

# Docker Logs

Show recent log output from a container.

## Usage

The argument is the container name or ID (e.g. `/docker-logs polaris-web`). If no argument, list running containers first and ask which one.

## Default

```bash
docker logs --tail 100 --timestamps $NAME
```

100 lines is enough to see a recent failure without overwhelming context.

## Follow mode

If the user wants live updates ("tail", "follow", "-f"):

```bash
docker logs --tail 50 --follow --timestamps $NAME
```

Run this in a background shell (`run_in_background: true`) so the conversation continues — the user can `ctrl-c` it or ask to stop.

## After running

- If the last few lines show errors/exceptions, surface them at the top of your summary — don't bury in the dump.
- If the log is noisy (one line per request), mention it and offer to filter (`| grep ERROR`, etc.).
- If the container has been up for days, the log may be huge — `--tail` caps this, but warn if older context is needed.

## Notes

- `--timestamps` adds ISO timestamps — essential for correlating with other logs.
- If the container doesn't exist, suggest `docker ps -a` to find stopped containers (logs still work on stopped containers).
