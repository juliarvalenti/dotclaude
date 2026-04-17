---
name: docker
description: Docker and Docker Compose operations — list containers, tail logs, exec into a shell, prune for disk space, bring a compose stack up or down. Use when the user asks about Docker containers or Docker Compose.
user_invocable: true
---

# Docker

Container and compose operations. Match the user's intent to the section below.

## List containers

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
```

Include stopped containers with `-a`. Group by compose project (first hyphen-segment of the name) if many.

## Logs

```bash
docker logs --tail 100 --timestamps <name>
```

If the user wants to follow, use `--follow` and run in a background shell so the conversation continues.

Surface errors/exceptions at the top of your summary. `--timestamps` adds ISO times — essential for correlating.

Logs work on stopped containers too.

## Shell into a container

Try bash first, fall back to sh:

```bash
docker exec -it <name> bash 2>/dev/null || docker exec -it <name> sh
```

**Interactive `-it` sessions don't work from non-interactive Bash tool calls.** Suggest the user run the command themselves via `! docker exec -it <name> bash`. For one-off non-interactive commands (reads, checks), run directly without `-it`.

## Prune (disk cleanup)

This is destructive — confirm each step.

1. Show current usage: `docker system df`
2. Stopped containers: `docker container prune -f`
3. Dangling images: `docker image prune -f`
4. **⚠️ Unused volumes** (deletes data from stopped DB containers): `docker volume prune -f` — confirm before running
5. Build cache: `docker builder prune -f` (usually safe)
6. Show delta: `docker system df`

Nuclear: `docker system prune -a --volumes`. Never run without explicit confirmation — walk through what it'll delete first.

## Compose: find the file

Walk up from cwd to find the nearest `docker-compose.yml` / `compose.yml`:

```bash
d=$(pwd)
FILE=""
while [ "$d" != "/" ]; do
  for name in docker-compose.yml docker-compose.yaml compose.yml compose.yaml; do
    [ -f "$d/$name" ] && FILE="$d/$name" && break 2
  done
  d=$(dirname "$d")
done
echo "${FILE:-no compose file found}"
```

Then `cd "$(dirname $FILE)"` before running compose commands.

## Compose: up

```bash
docker compose up -d
```

Always detached (`-d`) — foreground mode blocks the conversation. Pass through any user args (`--build`, service names).

Report status after: `docker compose ps`.

## Compose: down

```bash
docker compose down
```

- `-v` also removes named volumes — ⚠️ destroys data. Confirm before running if there's a DB.
- `--rmi all` also removes images.
- For "stop but keep around": `docker compose stop` (not teardown).

## Notes

- Prefer `docker compose` (space, plugin form) over legacy `docker-compose` (hyphen).
- If a container name isn't given, run `docker ps` and ask.
- If the user has many containers belonging to one project, `docker compose logs -f` is often better than per-container `docker logs`.
