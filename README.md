[English](README.md) | [中文](README.zh-CN.md)

# bark-notify-skill

A Claude Code plugin that pushes a notification to your iPhone via [Bark](https://github.com/Finb/Bark) when Claude Code finishes a turn, errors out, or asks for your approval.

## Why you'd want this

- Claude finished in 30 seconds. You noticed 12 minutes later.
- You went to make coffee. Claude has been silently waiting on your approval the whole time.
- Your terminal is in tmux, your phone is across the room — and you'd still like the push to arrive.
- You closed the lid and walked away. The push lands anyway.

## Requirements

- macOS with Claude Code installed
- iPhone with the [Bark](https://apps.apple.com/app/bark-customed-notifications/id1403753865) app installed and a push URL ready (`https://api.day.app/<your-device-key>`)

## Install

In any Claude Code session:

```text
/plugin marketplace add RUIIIOVO/bark-notify-skill
/plugin install bark-notify-skill@bark-notify-skill
/reload-plugins
```

## Configure

```text
/bark-notify-skill:bark-notify-setup
```

You will be asked for:

1. Your Bark push URL — `https://api.day.app/<device-key>`
2. Plain or **encrypted** mode (encrypted is recommended if your push URL is sensitive)
3. If encrypted: a 16-character key (must match your iPhone Bark settings: `AES128` / `CBC` / `pkcs7`)

Verify:

```text
/bark-notify-skill:bark-notify-test
```

## Notifications

The push **title** is always the project directory name. The **body** is differentiated per event:

| When | Body | Bark `level` |
|------|------|--------------|
| Claude finishes a turn normally | `✅ Claude Code 已完成` | `active` |
| Claude needs your approval (tool-permission prompt) | `🔔 Claude Code 等待操作` | `timeSensitive` |
| A turn ends with an error | `❌ Claude Code 出错` | `timeSensitive` |
| The session ends abnormally (crash, killed) | `⚠️ Claude Code 会话异常结束` | `timeSensitive` |
| You close the CLI yourself (`/exit`, `/clear`, `/logout`) | **silent** | — |
| Claude Code's idle-input 60s reminder | **silent** | — |

## tmux

The script emits a terminal bell after every notification. To surface it in tmux:

```bash
tmux set -g monitor-bell on
tmux set -g visual-bell on
```

## Uninstall

```text
/bark-notify-skill:bark-notify-uninstall
```

Removes the Bark hooks from `~/.claude/settings.json` and deletes the local script.

## Troubleshooting

Ask Claude in any session — the `bark-notify` skill will guide you. Or re-run setup to reconfigure.

## Privacy

- Your Bark URL and encryption key are stored locally in `~/.claude/` and never sent anywhere except your Bark server.
- Setup merges into your existing `~/.claude/settings.json` without overwriting other entries.
- In encrypted mode, the notification body is AES-128-CBC encrypted before leaving your Mac.

---

## For contributors

<details>
<summary>Local development</summary>

```bash
git clone https://github.com/RUIIIOVO/bark-notify-skill.git
cd bark-notify-skill
claude --plugin-dir .
```

Skills:

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `bark-notify` | Auto (model-invoked) | Guidance, setup decisions, troubleshooting |
| `bark-notify-setup` | `/bark-notify-skill:bark-notify-setup` | Interactive setup |
| `bark-notify-test` | `/bark-notify-skill:bark-notify-test` | Verification notification |
| `bark-notify-uninstall` | `/bark-notify-skill:bark-notify-uninstall` | Removes hook and script |

</details>

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## License

MIT
