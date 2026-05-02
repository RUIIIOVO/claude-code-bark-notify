---
name: bark-notify-uninstall
description: Remove Bark notification configuration from Claude Code on macOS, Linux, or Windows. Use when the user wants to uninstall, remove, or clean up Bark notifications, delete the Bark hook, or reset their Bark setup.
argument-hint: none
---

# Bark Notify Uninstall

Remove the Bark notification hook and script from the local Claude Code environment.

## Behavior

1. Detect the current OS (same method as setup: check working directory path or `$env:OS`).
2. Read `~/.claude/settings.json` if it exists.
3. Remove the `Stop`, `StopFailure`, `SessionEnd`, and `Notification` hooks that reference `claude-stop-bark.sh` (macOS/Linux) or `claude-stop-bark.ps1` (Windows).
   - Preserve all other hooks and settings.
   - If a hook event has other hooks besides the Bark one, keep those.
   - If a hook event does not exist, skip it.
   - If removing all Bark hooks leaves the `hooks` key empty, remove the `hooks` key entirely.
4. Delete the platform-specific script if it exists:
   - **macOS/Linux**: `~/.claude/claude-stop-bark.sh`
   - **Windows**: `%USERPROFILE%\.claude\claude-stop-bark.ps1` (resolve the absolute path)
5. Report what was removed to the user.

## Safety rules

- Never delete unrelated hooks or settings.
- If `~/.claude/settings.json` is malformed, stop and report the problem.
- Confirm with the user before removing files.
