[English](README.md) | [中文](README.zh-CN.md)

# bark-notify

A Claude Code plugin that pushes a notification to your iPhone via [Bark](https://github.com/Finb/Bark) when Claude Code finishes a turn, errors out, or asks for your approval.

## Why I built this

- In `vibe coding`, especially when juggling multiple projects, I often need to glance back at the terminal just to see whether Claude is done, has failed, or is stuck waiting for my approval.
- In an office, sound alerts are not always practical. And when you're looking at your phone while doing something else in parallel, that repeated “should I check the terminal again?” interruption gets annoying fast.
- Claude does have an official mobile notification path, but it is more aligned with the `Cowork` / `Dispatch` / `Remote Control` experience: install the Claude mobile app, enable notifications, keep the connection healthy, and in mainland China that usually also means keeping a VPN on all the time. If you're casually multitasking on your phone, that extra network friction gets annoying too.
- If you mainly use `Claude Code CLI`, the official customization surface is still mostly `hooks`; the official phone-push experience lives more in `Remote Control` / `Dispatch`, not as a lightweight phone notification path built into plain CLI usage.
- There are other workarounds too — Feishu or DingTalk bots, `iMessage`, email — but they are either heavy on setup, permission-gated, or too tied to one platform for this kind of frequent reminder.
- If you also use `ccswitch` to swap models, the desktop-first route is not always the smoothest fit. My own setup is `ccswitch` + Doubao voice input + this plugin, and the overall `vibe coding` loop feels much better.
- So I built this plugin to do one thing well: notify me when Claude is actually done or needs me. Bark is simple to set up, does not require an extra account, generates a key for you, and becomes usable as soon as you hand the URL to Claude.
- If privacy matters, you can turn on encryption or self-host Bark. At least for the notification layer, you do not have to route everything through a work chat tool or another third-party notification chain.

## Requirements

- macOS with Claude Code installed
- iPhone with the [Bark](https://apps.apple.com/app/bark-customed-notifications/id1403753865) app installed and a push URL ready (`https://api.day.app/<your-device-key>`)

## Install

In any Claude Code session:

```text
/plugin marketplace add RUIIIOVO/claude-code-bark-notify
/plugin install bark-notify@bark-notify
/reload-plugins
```

## Configure

```text
/bark-notify:bark-notify-setup
```

You will be asked for:

1. Your Bark push URL — `https://api.day.app/<device-key>`
2. Plain or **encrypted** mode (encrypted is recommended if your push URL is sensitive)
3. If encrypted: a 16-character key (must match your iPhone Bark settings: `AES128` / `CBC` / `pkcs7`)

Verify:

```text
/bark-notify:bark-notify-test
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
/bark-notify:bark-notify-uninstall
```

Removes the Bark hooks from `~/.claude/settings.json` and deletes the local script.

## Troubleshooting

Ask Claude in any session — the `bark-notify` plugin includes a guidance skill for setup and troubleshooting. Or re-run setup to reconfigure.

## Privacy

- Your Bark URL and encryption key are stored locally in `~/.claude/` and never sent anywhere except your Bark server.
- Setup merges into your existing `~/.claude/settings.json` without overwriting other entries.
- In encrypted mode, the notification body is AES-128-CBC encrypted before leaving your Mac.

---

## For contributors

<details>
<summary>Local development</summary>

```bash
git clone https://github.com/RUIIIOVO/claude-code-bark-notify.git
cd claude-code-bark-notify
claude --plugin-dir .
```

Skills:

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `bark-notify` | Auto (model-invoked) | Guidance, setup decisions, troubleshooting |
| `bark-notify-setup` | `/bark-notify:bark-notify-setup` | Interactive setup |
| `bark-notify-test` | `/bark-notify:bark-notify-test` | Verification notification |
| `bark-notify-uninstall` | `/bark-notify:bark-notify-uninstall` | Removes hook and script |

</details>

## Changelog

See [CHANGELOG.md](CHANGELOG.md) or [CHANGELOG.zh-CN.md](CHANGELOG.zh-CN.md).

## License

MIT
