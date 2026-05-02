---
name: bark-notify-test
description: Send a Bark verification notification for the current Claude Code Bark setup on macOS or Windows. Use when the user wants to test delivery, confirm encryption works, or check whether the configured completion notification is healthy.
argument-hint: optional project name
---

# Bark Notify Test

Send one verification notification using the user's local Bark script or Bark configuration.

## Behavior

1. Detect the current OS (same method as setup: check working directory path or `$env:OS`).
2. Check for the platform-specific script:
   - **macOS/Linux**: `~/.claude/claude-stop-bark.sh`
   - **Windows**: `%USERPROFILE%\.claude\claude-stop-bark.ps1` (resolve the absolute path)
3. If the script exists, use it for the test instead of inventing a second send path.
4. Pipe a `Stop` payload with the current or user-provided project name through the script:
   - macOS/Linux: `echo '{"hook_event_name":"Stop","cwd":"/path/to/project"}' | bash ~/.claude/claude-stop-bark.sh`
   - Windows: `echo '{"hook_event_name":"Stop","cwd":"C:\\path\\to\\project"}' | powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File "C:\Users\<username>\.claude\claude-stop-bark.ps1"`
5. Tell the user exactly what should appear on the phone:
   - title = project name
   - body = `✅ Claude Code 已完成`
6. If the notification fails, read `../bark-notify/references/troubleshooting.md` and debug from there.

## Notes

- Prefer a real push test over a dry run when the user explicitly asks for verification.
- For encrypted mode, the most common failure is mismatched Bark iPhone settings or wrong key.
- On Windows, if powershell.exe exits with a non-zero code, check that the absolute script path is correct and that the script file exists.
