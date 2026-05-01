---
name: bark-notify-setup
description: Set up Bark completion notifications for Claude Code on macOS by safely writing the local Bark sender script and merging a Stop hook into ~/.claude/settings.json. Use when the user explicitly asks to configure Bark for them or to directly install the completion notification flow.
argument-hint: [none]
version: 0.2.0
---

# Bark Notify Setup

Configure the current macOS Claude Code environment to send a Bark notification when Claude finishes a turn.

## Goals

After setup, the local machine should contain:
- `~/.claude/claude-stop-bark.sh`
- hooks in `~/.claude/settings.json` for `Stop`, `StopFailure`, `SessionEnd`, and `Notification` that all run that script asynchronously

## Required behavior

1. Confirm Bark prerequisites first.
   - Read `../bark-notify/references/bark-prereqs.md`.
   - Make sure the user has a Bark push URL.
   - Ask whether they want plain mode or encrypted mode.
   - If encrypted mode, require a 16-character key and remind them to match Bark app settings on iPhone.

2. Inspect current Claude config before writing.
   - Read `~/.claude/settings.json` if it exists.
   - Never overwrite unrelated settings.
   - Merge in the `Stop`, `StopFailure`, `SessionEnd`, and `Notification` hooks safely.

3. Use the config templates from `../bark-notify/references/config-patterns.md`.
   - Write a small shell script to `~/.claude/claude-stop-bark.sh`.
   - Parse `hook_event_name`, `reason`, and `cwd` from the stdin JSON payload using `python3 -c "..."` (do NOT use a python heredoc — it collides with the piped stdin payload). Do NOT read `$HOOK_EVENT` — Claude Code does not set that env var.
   - Branch the notification per the event-types table in `config-patterns.md`:
     - `Stop` → `✅ Claude Code 已完成`, level `active`
     - `Notification` → `🔔 Claude Code 等待操作`, level `timeSensitive`. **Filter:** parse `notification_type` from the payload; only send for `permission_prompt` and `elicitation_dialog`. For `idle_prompt`, `auth_success`, `elicitation_complete`, `elicitation_response` → `exit 0` silently. If `notification_type` is missing (older payload schema), fall back to checking `message`: skip when it contains `"waiting for your input"` or `"idle"`, otherwise send.
     - `StopFailure` → `❌ Claude Code 出错`, level `timeSensitive`
     - `SessionEnd` with `reason ∈ {clear, logout, prompt_input_exit}` → **`exit 0` immediately, no notification, no bell** (this is the user closing the CLI)
     - `SessionEnd` with any other reason → `⚠️ Claude Code 会话异常结束`, level `timeSensitive`
   - Do NOT set the `sound` field (audio cues opted out). Body emoji + `level` already differentiate events.
   - title = project name (basename of `cwd`); do NOT set the `icon` field (Cloudflare gates the official Claude icon; let Bark use its default).
   - End the script with `printf '\a'` (terminal bell) for tmux/terminal notifications. The user-initiated `SessionEnd` branch must exit before reaching the bell too.

4. Show the user what will change before applying it.
   - Mention both file paths.
   - Mention whether the Bark URL and encryption key will be stored locally.
   - Mention the differentiated bodies and the silent-on-user-exit behavior.

5. Validate after writing.
   - Ensure the script is executable.
   - Confirm all four hook commands (`Stop`, `StopFailure`, `SessionEnd`, `Notification`) point to `/bin/bash ~/.claude/claude-stop-bark.sh`.
   - Smoke-test by piping each event payload (`Stop`, `Notification`, `StopFailure`, `SessionEnd`+`reason=logout`, `SessionEnd`+`reason=other`) through the script and confirm the logout case exits without sending.
   - If the user wants, immediately hand off to `/bark-notify:bark-notify-test`.

## Safety rules

- Do not delete existing hooks unless the user explicitly asks.
- If there is already a `Stop` hook, merge carefully and explain the result.
- If JSON is malformed, stop and show the problem instead of forcing a write.
