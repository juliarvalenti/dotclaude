---
name: last-screenshot
description: View the most recent screenshot
user_invocable: true
---

Read the most recent screenshot file from `~/Screenshots/` (sorted by modification time) using the Read tool. Screenshots are PNG files named like `Screenshot YYYY-MM-DD at H.MM.SS PM.png`.

Steps:
1. Run: `ls -1 ~/Screenshots/Screenshot*.png 2>/dev/null | sort -r | head -1` to get the path (sort by filename, not mtime, since filenames contain timestamps)
2. Use the Read tool to view the image file at that path
3. Describe what you see to the user
