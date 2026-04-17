---
name: compose-up
description: Find the nearest docker-compose.yml and bring the stack up in detached mode. Use when the user says "compose up", "start the stack", "bring up the containers", or is in a project with a compose file.
user_invocable: true
---

# Compose Up

Start a Docker Compose stack.

## Steps

1. **Find the compose file** by walking up from the current directory:
   ```bash
   FILE=$(find . -maxdepth 3 -name "docker-compose.y*ml" -o -name "compose.y*ml" 2>/dev/null | head -1)
   if [ -z "$FILE" ]; then
     # walk up
     d=$(pwd)
     while [ "$d" != "/" ]; do
       for name in docker-compose.yml docker-compose.yaml compose.yml compose.yaml; do
         [ -f "$d/$name" ] && FILE="$d/$name" && break 2
       done
       d=$(dirname "$d")
     done
   fi
   echo "${FILE:-no compose file found}"
   ```

2. **If found, cd to the directory and start**:
   ```bash
   cd "$(dirname $FILE)"
   docker compose up -d
   ```

3. **Report status**:
   ```bash
   docker compose ps
   ```

## Arguments

The user's arguments pass through to `docker compose up`:

- `/compose-up` → `docker compose up -d`
- `/compose-up --build` → `docker compose up -d --build` (rebuild images)
- `/compose-up web db` → `docker compose up -d web db` (only those services)

## Notes

- Always `-d` (detached) by default — foreground mode blocks the conversation.
- If services fail to start, check logs via `/docker-logs <service>`.
- Modern Docker has `docker compose` (space) as the plugin; old installs used `docker-compose` (hyphen). Prefer the plugin form.
