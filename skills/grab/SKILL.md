---
name: grab
description: Read the most recent thing the user captured — clipboard, download, or screenshot. Also writes to clipboard. Use when the user says "what's on my clipboard", "look at what I copied", "my last download", "my last screenshot", "copy this", "put X on the clipboard".
user_invocable: true
---

# Grab

Read the user's most recent captured content from clipboard / Downloads / Screenshots, or write to clipboard. macOS only.

## Which source?

Match the user's language:
- "clipboard", "copied", "what's on my clipboard" → **clipboard**
- "download", "what I just downloaded", "the file I grabbed" → **downloads**
- "screenshot", "what I just captured", "the screen grab" → **screenshots**
- Ambiguous ("look at what I grabbed", "my last thing") → ask, or pick the most recently modified across all three

## Clipboard — read

```bash
pbpaste
```

If empty, say so. If binary/weird, note it rather than guessing.

## Clipboard — write

```bash
printf '%s' "CONTENT" | pbcopy
```

Use `printf '%s'` (not `echo`) — no trailing newline. For multi-line content:

```bash
pbcopy <<'EOF'
multi-line
content here
EOF
```

Confirm with one short sentence ("Copied — N lines" / "Copied the URL"). Don't re-print the full content.

## Last download

```bash
ls -t ~/Downloads/ | grep -v -E '^\.|\.download$|\.crdownload$' | head -1
```

Then use the Read tool — handles text, PDFs, images natively. For archives/binaries Read can't open:

```bash
ls -lh ~/Downloads/<file>
file ~/Downloads/<file>
```

Describe what it is and offer to extract / inspect.

## Last screenshot

Filenames in `~/Screenshots/` encode the timestamp, so sort by filename (not mtime — more reliable):

```bash
ls -1 ~/Screenshots/Screenshot*.png 2>/dev/null | sort -r | head -1
```

Use the Read tool on the resulting path — Claude can view images directly.

## Notes

- If the expected location is empty (no downloads, no screenshots, empty clipboard), say so — don't substitute from another source without asking.
- Browser partial downloads (`.download`, `.crdownload`) are skipped.
