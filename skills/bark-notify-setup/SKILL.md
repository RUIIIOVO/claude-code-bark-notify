---
name: bark-notify-setup
description: Set up Bark completion notifications for Claude Code on macOS or Windows by safely writing the local Bark sender script and merging hooks into ~/.claude/settings.json. Use when the user explicitly asks to configure Bark for them or to directly install the completion notification flow.
argument-hint: none
---

# Bark Notify Setup

Configure the current Claude Code environment (macOS, Linux, or Windows) to send a Bark notification when Claude finishes a turn.

## Goals

After setup, the local machine should contain:
- **macOS/Linux**: `~/.claude/claude-stop-bark.sh`
- **Windows**: `%USERPROFILE%\.claude\claude-stop-bark.ps1`
- hooks in `~/.claude/settings.json` for `Stop`, `StopFailure`, `SessionEnd`, and `Notification` that all run that script asynchronously

## Required behavior

0. **Detect the current OS first.**
   - Run `uname -s 2>/dev/null` (bash) or check if the working directory path starts with a drive letter.
   - If `$env:OS` is `Windows_NT` or the path looks like `C:\...` or `D:\...`, treat as Windows.
   - Otherwise treat as macOS/Linux.
   - Read `../bark-notify/references/config-patterns.md` to get the platform-specific hook command and script template.

1. **Confirm Bark prerequisites.**
   - Read `../bark-notify/references/bark-prereqs.md`.
   - Make sure the user has a Bark push URL.
   - Ask whether they want plain mode or encrypted mode.
   - If encrypted mode, require a 16-character key and remind them to match Bark app settings on iPhone.

2. **Inspect current Claude config before writing.**
   - Read `~/.claude/settings.json` if it exists.
   - Never overwrite unrelated settings.
   - Merge in the `Stop`, `StopFailure`, `SessionEnd`, and `Notification` hooks safely.

3. **Write the sender script using the platform template from `../bark-notify/references/config-patterns.md`.**

   ### macOS / Linux
   - Write `~/.claude/claude-stop-bark.sh`.
   - Parse `hook_event_name`, `reason`, and `cwd` from stdin JSON using `python3 -c "..."` (do NOT use a python heredoc — it collides with the piped stdin payload). Do NOT read `$HOOK_EVENT` — Claude Code does not set that env var.
   - Hook command in settings.json: `/bin/bash ~/.claude/claude-stop-bark.sh`

   ### Windows
   - Write `%USERPROFILE%\.claude\claude-stop-bark.ps1`.
   - Resolve the absolute home path first (e.g. run `$env:USERPROFILE` in PowerShell or read it from the environment) and embed it as a literal path.
   - Use the PowerShell plain/encrypted template exactly as shown in `config-patterns.md`.
   - Hook command in settings.json: `powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File "C:\Users\<username>\.claude\claude-stop-bark.ps1"` — use the resolved absolute path, not `%USERPROFILE%`.
   - Do NOT run `chmod` on Windows — PS1 files do not need executable bits.

   ### Notification branching (both platforms)
   - `Stop` → `✅ Claude Code 已完成`, level `active`
   - `Notification` → `🔔 Claude Code 等待操作`, level `timeSensitive`. **Filter:** only send for `notification_type` = `permission_prompt` or `elicitation_dialog`; exit silently for `idle_prompt`, `auth_success`, `elicitation_complete`, `elicitation_response`. If `notification_type` is missing, fall back to `message`: skip when it contains `"waiting for your input"` or `"idle"`, otherwise send.
   - `StopFailure` → `❌ Claude Code 出错`, level `timeSensitive`
   - `SessionEnd` with `reason ∈ {clear, logout, prompt_input_exit}` → **exit immediately, no notification, no bell**
   - `SessionEnd` with any other reason → `⚠️ Claude Code 会话异常结束`, level `timeSensitive`
   - Do NOT set the `sound` field.
   - title = project name (basename of `cwd`); do NOT set the `icon` field.
   - End the script with a terminal bell (`printf '\a'` on bash; `[Console]::Write([char]7)` on PowerShell). The user-initiated `SessionEnd` branch must exit before the bell.

4. **Show the user what will change before applying it.**
   - Mention both file paths.
   - Mention whether the Bark URL and encryption key will be stored locally.
   - Mention the differentiated bodies and the silent-on-user-exit behavior.

5. **Validate after writing.**
   - macOS/Linux: ensure the script is executable (`chmod +x`).
   - Windows: no chmod needed.
   - Confirm all four hook commands (`Stop`, `StopFailure`, `SessionEnd`, `Notification`) point to the correct script.
   - Smoke-test by piping each event payload (`Stop`, `Notification`, `StopFailure`, `SessionEnd`+`reason=logout`, `SessionEnd`+`reason=other`) through the script:
     - macOS/Linux: `echo '...' | bash ~/.claude/claude-stop-bark.sh`
     - Windows: `echo '...' | powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File "C:\Users\<username>\.claude\claude-stop-bark.ps1"`
   - Confirm the logout case exits without sending.
   - If the user wants, immediately hand off to `/bark-notify:bark-notify-test`.

## Safety rules

- Do not delete existing hooks unless the user explicitly asks.
- If there is already a `Stop` hook, merge carefully and explain the result.
- If JSON is malformed, stop and show the problem instead of forcing a write.
