---
name: docker-shell
description: Exec into a running Docker container's shell for interactive debugging. Use when the user says "shell into X", "get me inside X", "exec into X", or needs to poke around a container's filesystem.
user_invocable: true
---

# Docker Shell

Open an interactive shell inside a running container.

## Usage

The argument is the container name or ID (e.g. `/docker-shell polaris-web`). If no argument, list running containers and ask.

## Auto-detect the shell

Most images ship with `/bin/sh`; many (Debian/Ubuntu-based) also have `/bin/bash`. Try bash first, fall back to sh:

```bash
docker exec -it $NAME bash 2>/dev/null || docker exec -it $NAME sh
```

## Non-interactive variant

If the user wants to run a one-off command inside the container (not an interactive session):

```bash
docker exec $NAME <command>
```

Example: `docker exec polaris-web cat /app/.env`

## Important

`docker exec -it` requires a TTY and won't work from non-interactive Bash tool calls. When the user asks for an interactive shell, **suggest they run the command themselves** via `! docker exec -it <name> bash` at the Claude Code prompt — don't try to run the interactive form yourself.

For one-off non-interactive commands (file reads, checking process state, etc.), run them directly.

## After running

- If the container exits with "executable file not found" for bash, retry with sh.
- Ephemeral containers may not have basic tools — if `ls` / `cat` aren't there, the image is probably distroless or scratch-based.
