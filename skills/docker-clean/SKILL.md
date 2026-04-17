---
name: docker-clean
description: Reclaim disk space by pruning stopped Docker containers, dangling images, and unused volumes. Use when the user says "clean docker", "prune docker", "free up docker space", or is running low on disk.
user_invocable: true
---

# Docker Clean

Reclaim Docker disk space. This is destructive — confirm before doing each step.

## Steps

1. **Show current disk usage**:
   ```bash
   docker system df
   ```
   Report the totals so the user sees what's at stake.

2. **Prune stopped containers**:
   ```bash
   docker container prune -f
   ```

3. **Prune dangling images** (images with no tag, left over from rebuilds):
   ```bash
   docker image prune -f
   ```

4. **Prune unused volumes** — ⚠️ this deletes volumes not attached to any container. If the user has stopped a DB container but wants to keep the data, this will nuke it:
   ```bash
   docker volume prune -f
   ```
   **Confirm before running this step.** Ask: "Prune unused volumes too? This deletes any volume not attached to a running container — including data from stopped DB containers."

5. **Optional: prune build cache** (separate from images):
   ```bash
   docker builder prune -f
   ```
   Usually safe. Frees cached build layers.

6. **Show final usage**:
   ```bash
   docker system df
   ```
   Report the delta.

## Nuclear option

`docker system prune -a --volumes` removes everything not in use, including unused images (even tagged ones) and all unused volumes. **Never run this without explicit user confirmation** — if they say "nuke everything" or "system prune", walk them through what it'll delete first.

## Notes

- `docker container prune` only touches **stopped** containers. Running ones are untouched.
- "Dangling" images are ones with `<none>:<none>` tags — leftovers from rebuilds. Safe to delete.
- If the user is actively mid-project, check `docker ps -a` first — a recently stopped container they might restart shouldn't be pruned.
