---
name: disk
description: Disk usage utilities — overall free space, biggest directories, find space hogs. Use when the user asks "why is my laptop full", "what's taking up space", "disk usage", or runs low on disk.
user_invocable: true
---

# Disk

Observational disk usage utilities. This skill **reports** — the user decides what to do with what you find. Never delete without explicit, specific instruction from the user.

## Overall free space

```bash
df -h /
```

Report free / used / total for the root volume. Flag if < 10% free.

## Biggest common suspects

On macOS dev machines, check the usual culprits:

```bash
du -sh ~/Library/Caches 2>/dev/null
du -sh ~/Library/Developer 2>/dev/null
du -sh ~/.docker 2>/dev/null
du -sh ~/Documents/GitHub 2>/dev/null
du -sh ~/Downloads 2>/dev/null
du -sh ~/.Trash 2>/dev/null
```

Report as a sorted list, biggest first.

## node_modules / venv / dist sweep

Find regeneratable build artifacts across `~/Documents/GitHub/`:

```bash
find ~/Documents/GitHub -maxdepth 4 -type d \( -name "node_modules" -o -name ".venv" -o -name "dist" -o -name ".next" -o -name ".turbo" \) -prune -exec du -sh {} \; 2>/dev/null | sort -hr | head -20
```

Report the top offenders with their sizes and paths. Do **not** suggest deletion — the user knows which projects they're actively working in and which are dormant.

## Docker disk

```bash
docker system df
```

Shows images, containers, volumes, build cache. If large, mention that `/docker` has a prune section the user can invoke if they want to reclaim space.

## Drill into a specific dir

```bash
du -sh $DIR/* 2>/dev/null | sort -hr | head -10
```

Top 10 biggest entries inside a directory. Repeat on the biggest child to drill further.

## Reporting rules

- Present findings as observations, not recommendations to delete.
- If the user asks "what can I delete", describe the category (build artifacts, caches, downloads) and let them pick — don't enumerate a "safe to clear" list.
- If the user explicitly names something to delete ("rm that node_modules", "clear my Downloads folder"), confirm the path and size before running `rm`.
- Caches and build artifacts can regenerate, but deleting a cache mid-build breaks things. Deleting a Downloads folder may lose files the user needs. Treat all delete requests as requiring confirmation on specifics.

## Notes

- `du` can be slow on large trees — cap with `-d 1` / `-maxdepth 1` / `-maxdepth 2` for first pass.
- `find ... -prune` stops descent into matched dirs — important for node_modules scans so it doesn't recurse inside them.
- `df` output doesn't always update instantly after a delete — Spotlight or Time Machine may still hold references for a minute.
