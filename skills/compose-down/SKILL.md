---
name: compose-down
description: Find the nearest docker-compose.yml and bring the stack down. Use when the user says "compose down", "stop the stack", "shut down the containers", or wants to tear down a dev environment.
user_invocable: true
---

# Compose Down

Stop a Docker Compose stack.

## Steps

1. **Find the compose file** (walk up from cwd — same as compose-up):
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

2. **Stop**:
   ```bash
   cd "$(dirname $FILE)"
   docker compose down
   ```

## Arguments

- `/compose-down` → `docker compose down`
- `/compose-down -v` → also remove named volumes (⚠️ destroys data). **Confirm with user before running** — if they have a DB in the stack, this wipes it.
- `/compose-down --rmi all` → also remove images the stack built/pulled.

## Notes

- `docker compose down` stops + removes containers, networks. Volumes and images survive by default.
- For just "stop but keep around": `docker compose stop` (not handled by this skill — suggest it if the user doesn't want full teardown).
