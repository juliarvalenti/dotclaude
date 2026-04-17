---
name: kill-port
description: Kill whatever process is listening on a given TCP port. Use when the user says "kill port N", "something's on N", "free up port N", or is trying to start a dev server and the port is taken.
user_invocable: true
---

# Kill port

Kill the process (or processes) listening on a TCP port.

## Usage

The user's argument is the port number (e.g. `/kill-port 3000`).

## Steps

1. **Check what's there first** — don't kill blind:
   ```bash
   lsof -iTCP:$PORT -sTCP:LISTEN -P -n
   ```
   Shows command, PID, and process owner. Report this to the user before killing.

2. **Kill it**:
   ```bash
   lsof -tiTCP:$PORT -sTCP:LISTEN | xargs -r kill
   ```
   Use plain `kill` (SIGTERM) first. Most well-behaved dev servers will exit cleanly.

3. **Verify it's gone**:
   ```bash
   lsof -iTCP:$PORT -sTCP:LISTEN -P -n
   ```
   Should return nothing.

4. **If it's still there**, escalate to `kill -9`:
   ```bash
   lsof -tiTCP:$PORT -sTCP:LISTEN | xargs -r kill -9
   ```

## Notes

- If nothing is listening on the port, say so — don't invent a result.
- If the process is owned by a different user (shouldn't happen on a personal machine), flag it rather than `sudo kill` automatically.
- Ports < 1024 require root to bind — killing one that's running as root requires `sudo`. Ask before.
