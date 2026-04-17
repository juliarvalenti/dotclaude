---
name: clipboard
description: Read or write the macOS clipboard. Use when the user says "what's on my clipboard", "look at what I copied", "read my clipboard", or asks to put content on the clipboard ("copy this to my clipboard", "put X on the clipboard").
user_invocable: true
---

# Clipboard

Read or write the macOS system clipboard.

## Read

```bash
pbpaste
```

Shows whatever is currently on the clipboard. If empty, say so — don't guess.

## Write

```bash
printf '%s' "CONTENT" | pbcopy
```

Use `printf '%s'` (not `echo`) to avoid adding a trailing newline. For multi-line content, use a heredoc:

```bash
pbcopy <<'EOF'
multi-line
content here
EOF
```

After writing, confirm with a short sentence naming what was copied (don't re-print the full content).

## Notes

- `pbpaste` output may contain binary or weird characters. If it doesn't look like text, note that.
- On macOS only — this skill is not portable to Linux.
