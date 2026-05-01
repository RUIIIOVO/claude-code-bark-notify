# Config patterns

This reference contains the supported Bark hook patterns.

## Stop hook shape

Merge this structure into `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/claude-stop-bark.sh",
            "async": true
          }
        ]
      }
    ],
    "StopFailure": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/claude-stop-bark.sh",
            "async": true
          }
        ]
      }
    ],
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/claude-stop-bark.sh",
            "async": true
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/bin/bash ~/.claude/claude-stop-bark.sh",
            "async": true
          }
        ]
      }
    ]
  }
}
```

Always merge instead of replacing unrelated config.

## Event types

The script must differentiate notifications so the user can tell at a glance what happened. Read `hook_event_name` and `reason` from the JSON payload on **stdin** (not from any environment variable — Claude Code does not set `HOOK_EVENT`).

| Hook event | `reason` (SessionEnd only) | Notification body | `level` |
|------------|---------------------------|-------------------|---------|
| `Stop` | — | `✅ Claude Code 已完成` | `active` |
| `Notification` | — | `🔔 Claude Code 等待操作` | `timeSensitive` |
| `StopFailure` | — | `❌ Claude Code 出错` | `timeSensitive` |
| `SessionEnd` | `other` (or unset) | `⚠️ Claude Code 会话异常结束` | `timeSensitive` |
| `SessionEnd` | `clear` / `logout` / `prompt_input_exit` | **skip — no notification, no bell** | — |

`Notification` fires for **multiple reasons**, not all of them actionable:
- `notification_type: "permission_prompt"` — Claude needs the user to approve a tool call (1/2/3 prompt). **Send.**
- `notification_type: "idle_prompt"` — input box has been idle for ~60s. **Skip silently** — it's just nagging.
- `notification_type: "auth_success"` / `"elicitation_complete"` / `"elicitation_response"` — informational, no user action required. **Skip silently.**
- `notification_type: "elicitation_dialog"` — the user is being asked something. **Send.**
- Unknown / missing `notification_type` — fall back to sniffing the legacy `message` field; skip if it contains `"waiting for your input"` or `"idle"`, otherwise send (be conservative — better to over-notify than miss a permission prompt).

Do not set the `sound` field. The body emoji + Bark's `level` already differentiate events; an extra audio cue is redundant and was explicitly opted out of.

The last row matters: when the user explicitly closes the CLI, runs `/clear`, or `/logout`, the script must `exit 0` before doing any work. Otherwise every CLI close fires a stale "completed" push.

## Plain Bark script pattern

Plain mode uses the Bark push URL directly and sends readable title/body content.

Script responsibilities:
- read hook payload from stdin
- parse `hook_event_name`, `reason`, `notification_type`, `message`, and `cwd` from the JSON payload (use `python3 -c "..."`, not a heredoc — a heredoc on stdin collides with the piped payload)
- derive project name from the `cwd` basename
- branch on the event-types table above; for user-initiated `SessionEnd`, exit immediately
- send title = project name
- send body and level based on the chosen branch
- omit the `sound` field (no audio cue requested)
- omit the `icon` field — Bark's default icon is fine and any external URL (e.g. `claude.ai/images/...`) is gated behind Cloudflare human verification, which Bark's image fetch cannot pass

## Encrypted Bark script pattern

Encrypted mode follows the same event-type branching, but sends a ciphertext payload plus `iv`.

Script responsibilities:
- branch on event/reason first; on user-initiated `SessionEnd`, exit before any encryption work
- build plaintext JSON with title/body/group/level/isArchive (no `sound`, no `icon` — see plain-mode note above)
- generate a random 16-character `iv`
- convert key and `iv` to hex for OpenSSL
- encrypt with `AES-128-CBC`
- base64 the ciphertext
- send `ciphertext` and `iv` to the Bark push URL

## Terminal bell for tmux / terminal notifications

After sending the Bark push, the script should output a terminal bell character (`\a`). This enables tmux and terminal apps to detect activity and notify the user — including on Ctrl+C interrupts where Claude Code hooks do not fire.

Add this line at the end of the script:

```bash
printf '\a'
```

For tmux, users should enable bell monitoring:

```bash
tmux set -g monitor-bell on
tmux set -g visual-bell on
```

Or use activity monitoring as a fallback:

```bash
tmux set -g monitor-activity on
tmux set -g visual-activity on
```

This way, even when a hook event does not fire (e.g., mid-turn Ctrl+C), tmux still detects the pane activity and notifies the user.

## Notification icon

Do not set the `icon` field. The official `claude.ai/images/claude_app_icon.png` is gated behind Cloudflare human-verification, so Bark cannot fetch it and falls back to a broken-image placeholder. Letting Bark use its own default icon looks better than a failed remote fetch.
