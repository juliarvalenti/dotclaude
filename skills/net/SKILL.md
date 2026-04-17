---
name: net
description: Network utilities — show what's on a port, kill a port, local/public IP, DNS lookup. Use when the user asks about ports ("what's on 3000", "kill port 8080"), IPs ("what's my IP"), or DNS ("what does example.com resolve to").
user_invocable: true
---

# Net

Grab-bag of network utilities. Match the user's intent to the section.

## What's on a port (listening)

```bash
lsof -iTCP:$PORT -sTCP:LISTEN -P -n
```

Shows command, PID, user, address. Report the process to the user — they may want `/net kill <port>` after, or may just be curious.

## Kill a port

1. **Show what's there first** — don't kill blind:
   ```bash
   lsof -iTCP:$PORT -sTCP:LISTEN -P -n
   ```

2. **Kill it** (SIGTERM first):
   ```bash
   lsof -tiTCP:$PORT -sTCP:LISTEN | xargs -r kill
   ```

3. **Verify gone**:
   ```bash
   lsof -iTCP:$PORT -sTCP:LISTEN -P -n
   ```
   Should return nothing.

4. **If still there**, escalate:
   ```bash
   lsof -tiTCP:$PORT -sTCP:LISTEN | xargs -r kill -9
   ```

Ports < 1024 may need `sudo`. Ask before.

## All listening ports

```bash
lsof -iTCP -sTCP:LISTEN -P -n
```

Useful when debugging "something's using a port I need" without knowing which port.

## Local IPs

```bash
ipconfig getifaddr en0   # Wi-Fi (most common on Mac)
ifconfig | grep "inet " | grep -v 127.0.0.1
```

Report the Wi-Fi IP prominently; list other interfaces as secondary.

## Public IP

```bash
curl -s ifconfig.me
```

Or `curl -s https://api.ipify.org` as a fallback if ifconfig.me is down.

## DNS lookup

```bash
dig +short $DOMAIN
```

For richer detail:
```bash
dig $DOMAIN
```

For reverse lookup:
```bash
dig -x $IP +short
```

`host` works too but `dig` output is more structured.

## Notes

- If nothing is listening on a port the user expected to be in use, say so — don't invent.
- When reporting IPs, note whether you're on Wi-Fi vs Ethernet if multiple interfaces are active.
- `lsof` may be slow on busy systems — if it hangs more than a few seconds, the user's firewall or network state may be misbehaving.
