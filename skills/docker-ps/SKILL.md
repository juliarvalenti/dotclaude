---
name: docker-ps
description: Show running Docker containers in a readable one-line-per-container format. Use when the user asks "what's running", "show containers", "docker ps", or needs to see container state before acting.
user_invocable: true
---

# Docker PS

List running (and optionally stopped) containers.

## Running containers

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
```

## All containers (including stopped)

Pass `-a` or `--all` as argument:

```bash
docker ps -a --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
```

## After running

- If no containers: say "No containers running" (or "No containers at all" for `-a`).
- Note anything unexpected — containers that have been up for weeks, exited with non-zero, or are restarting in a loop.
- If many containers belong to a `docker-compose.yml` project, group by project (the first hyphen-segment of the container name).
