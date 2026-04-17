---
name: notify
description: Send a macOS system notification from the agent. Use when a long-running task finishes, when the agent hits a blocker and needs user input, or when the user explicitly asks to be pinged ("notify me when done", "let me know when X finishes").
user_invocable: true
---

# Notify

Send a macOS notification so the user gets a ping even if they've switched apps. Built for the agent loop — when work takes a while, the user context-switches, and you need to reach them.

## Basic notification

```bash
osascript -e 'display notification "MESSAGE" with title "TITLE"'
```

Example:
```bash
osascript -e 'display notification "Build finished — 2 type errors" with title "Claude Code"'
```

## With subtitle and sound

```bash
osascript -e 'display notification "Details here" with title "Main title" subtitle "Optional subtitle" sound name "Glass"'
```

Available sound names include: `Glass`, `Ping`, `Pop`, `Basso`, `Blow`, `Bottle`, `Frog`, `Funk`, `Hero`, `Morse`, `Purr`, `Sosumi`, `Submarine`, `Tink`.

Use sound sparingly — default to silent for routine completion, use sound only when user input is blocked or something actually broke.

## When to notify

**Good reasons** (task-ending, agent-initiated):
- Long build / test suite finished — report pass/fail in the message
- Deploy completed — include the target (staging vs prod)
- Hit a blocker that requires user input — "Need your decision on X"
- Background agent (running in parallel) finished its work
- Long migration / data job done

**Bad reasons** (noise):
- Every commit
- Every file save
- Routine progress ("still working...")
- Things the user is actively watching — they don't need a ping for something in their current window

## Tone

Notifications are read in two seconds from a peripheral glance. Make them count:

- **Title**: what finished or needs attention ("Build done", "Deploy failed", "Need input")
- **Body**: the outcome or the ask ("All 47 tests passed", "2 type errors in web app", "Choose: merge squash or merge commit?")
- **No** emoji, no flair, no conversational tone. It's a status line, not a Slack message.

## Integration with other skills

- Long `/precommit` run → notify on completion
- `/ship` watching `gh pr checks --watch` → notify on check completion
- Any background agent (Agent tool with `run_in_background: true`) → notify on completion
- Deploy skill (if the project has one) → notify on deploy finish

## Notes

- First time `osascript` runs, macOS may prompt for notification permissions. User has to grant once.
- Notifications are best-effort — if Do Not Disturb / Focus is on, they'll be queued or suppressed.
- `display alert` is a different beast (blocking modal). Don't use — the agent can't clear it and it locks the user's attention.
