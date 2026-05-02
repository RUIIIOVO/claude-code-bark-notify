# Config patterns

This reference contains the supported Bark hook patterns for macOS/Linux and Windows.

## Platform detection

Before writing any files, detect the current OS:

- Run `uname -s 2>/dev/null` (bash) or check `$env:OS` (PowerShell).
- If the result is `Windows_NT`, or the working directory path starts with a drive letter (`C:\`, `D:\`, etc.), use the **Windows** path below.
- Otherwise use the **macOS/Linux** path.

---

## macOS / Linux — hook shape

Script path: `~/.claude/claude-stop-bark.sh`

Hook command: `/bin/bash ~/.claude/claude-stop-bark.sh`

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

---

## Windows — hook shape

Script path: `%USERPROFILE%\.claude\claude-stop-bark.ps1`

Resolve the **absolute path** at setup time (e.g. `C:\Users\Alice\.claude\claude-stop-bark.ps1`) and embed it literally in the hook command. Do not use `%USERPROFILE%` as a variable in the JSON — resolve it first.

Hook command (replace the path with the actual absolute path):

```
powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File "C:\Users\<username>\.claude\claude-stop-bark.ps1"
```

Merge this structure into `%USERPROFILE%\.claude\settings.json` (same file as `~/.claude/settings.json`):

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File \"C:\\Users\\<username>\\.claude\\claude-stop-bark.ps1\"",
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
            "command": "powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File \"C:\\Users\\<username>\\.claude\\claude-stop-bark.ps1\"",
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
            "command": "powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File \"C:\\Users\\<username>\\.claude\\claude-stop-bark.ps1\"",
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
            "command": "powershell.exe -ExecutionPolicy Bypass -NoProfile -NonInteractive -File \"C:\\Users\\<username>\\.claude\\claude-stop-bark.ps1\"",
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

## Plain Bark script pattern (macOS/Linux — bash)

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

## Plain Bark script pattern (Windows — PowerShell)

Use this template when generating `claude-stop-bark.ps1` on Windows.

Key differences from the bash version:
- Read stdin with `[System.IO.StreamReader]::new([Console]::OpenStandardInput(), [Text.Encoding]::UTF8)` — this handles the UTF-8 JSON payload correctly regardless of the system's console code page.
- Parse JSON with PowerShell's native `ConvertFrom-Json` — no `python3` dependency.
- Send HTTP with `Invoke-RestMethod` — no `curl` dependency.
- Build emoji with `[char]::ConvertFromUtf32()` — avoids PS1 file encoding issues; the values are constructed at runtime.
- Terminal bell: `[Console]::Write([char]7)` (equivalent to bash `printf '\a'`).
- No `chmod` needed — `.ps1` files do not need executable bits.

```powershell
$reader = [System.IO.StreamReader]::new([Console]::OpenStandardInput(), [Text.Encoding]::UTF8)
$payloadRaw = $reader.ReadToEnd()
if ([string]::IsNullOrWhiteSpace($payloadRaw)) { exit 0 }
try { $d = $payloadRaw | ConvertFrom-Json } catch { exit 0 }

$hookEvent = $d.hook_event_name
$reason    = $d.reason
$notifType = $d.notification_type
$message   = $d.message
$cwd       = $d.cwd

$project = if ($cwd) { Split-Path -Leaf $cwd } else { 'Claude Code' }

$BARK_URL = 'https://api.day.app/DEVICE-KEY/'

$body = ''; $level = ''

switch ($hookEvent) {
  'Stop' {
    $body  = "$([char]::ConvertFromUtf32(0x2705)) Claude Code 已完成"
    $level = 'active'
  }
  'StopFailure' {
    $body  = "$([char]::ConvertFromUtf32(0x274C)) Claude Code 出错"
    $level = 'timeSensitive'
  }
  'SessionEnd' {
    if ($reason -in @('clear','logout','prompt_input_exit')) { exit 0 }
    $body  = "$([char]::ConvertFromUtf32(0x26A0))$([char]0xFE0F) Claude Code 会话异常结束"
    $level = 'timeSensitive'
  }
  'Notification' {
    if ($notifType) {
      if ($notifType -in @('permission_prompt','elicitation_dialog')) {
        $body  = "$([char]::ConvertFromUtf32(0x1F514)) Claude Code 等待操作"
        $level = 'timeSensitive'
      } else { exit 0 }
    } else {
      if ($message -match 'waiting for your input|idle') { exit 0 }
      $body  = "$([char]::ConvertFromUtf32(0x1F514)) Claude Code 等待操作"
      $level = 'timeSensitive'
    }
  }
  default { exit 0 }
}

if (-not $body) { exit 0 }

$t = [Uri]::EscapeDataString($project)
$b = [Uri]::EscapeDataString($body)
try {
  Invoke-RestMethod -Uri "${BARK_URL}${t}/${b}?level=${level}&isArchive=1" -Method Get -TimeoutSec 10 | Out-Null
} catch { }

[Console]::Write([char]7)
```

## Encrypted Bark script pattern (macOS/Linux — bash)

Encrypted mode follows the same event-type branching, but sends a ciphertext payload plus `iv`.

Script responsibilities:
- branch on event/reason first; on user-initiated `SessionEnd`, exit before any encryption work
- build plaintext JSON with title/body/group/level/isArchive (no `sound`, no `icon` — see plain-mode note above)
- generate a random 16-character `iv`
- convert key and `iv` to hex for OpenSSL
- encrypt with `AES-128-CBC`
- base64 the ciphertext
- send `ciphertext` and `iv` to the Bark push URL

## Encrypted Bark script pattern (Windows — PowerShell)

Same event-type branching as plain mode. Replace the HTTP send block with:

```powershell
$BARK_KEY = 'YOUR-16-CHAR-KEY'

# Generate random 16-char IV from alphanumeric characters
$chars = (48..57) + (65..90) + (97..122)
$ivStr = -join (1..16 | ForEach-Object { [char](Get-Random -InputObject $chars) })

$plainObj = [ordered]@{ title = $project; body = $body; group = 'Claude Code'; level = $level; isArchive = 1 }
$plaintext = ConvertTo-Json $plainObj -Compress

$aes = [System.Security.Cryptography.Aes]::Create()
$aes.Mode    = [System.Security.Cryptography.CipherMode]::CBC
$aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7
$aes.KeySize = 128
$aes.Key     = [Text.Encoding]::UTF8.GetBytes($BARK_KEY)
$aes.IV      = [Text.Encoding]::UTF8.GetBytes($ivStr)

$enc          = $aes.CreateEncryptor()
$plainBytes   = [Text.Encoding]::UTF8.GetBytes($plaintext)
$cipherBytes  = $enc.TransformFinalBlock($plainBytes, 0, $plainBytes.Length)
$ciphertext   = [Convert]::ToBase64String($cipherBytes)

$jsonBody = ConvertTo-Json @{ ciphertext = $ciphertext; iv = $ivStr } -Compress
try {
  Invoke-RestMethod -Uri $BARK_URL -Method Post -Body $jsonBody -ContentType 'application/json' -TimeoutSec 10 | Out-Null
} catch { }
```

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
