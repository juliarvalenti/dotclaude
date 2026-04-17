---
name: last-download
description: Read or describe the most recently downloaded file in ~/Downloads/. Use when the user says "look at my last download", "read the file I just downloaded", "what did I just download", or references a download without naming the file.
user_invocable: true
---

# Last download

Read the most recent file in `~/Downloads/` using the Read tool.

## Steps

1. Find the most recent file (by modification time, excluding `.DS_Store` and partial downloads):
   ```bash
   ls -t ~/Downloads/ | grep -v -E '^\.|\.download$|\.crdownload$' | head -1
   ```

2. Use the Read tool to view it. Works for text, PDFs, images — Read handles each format.

3. For file types Read can't handle (archives, binaries), stat the file and describe:
   ```bash
   ls -lh ~/Downloads/<file>
   file ~/Downloads/<file>
   ```
   Offer to extract / inspect further if it's an archive (zip, tar, dmg).

## Notes

- If `~/Downloads/` is empty, say so — don't substitute a different directory.
- Browser partial downloads end in `.download` or `.crdownload` — skip those.
