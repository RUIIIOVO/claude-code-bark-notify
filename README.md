[English](README.md) | [中文](README.zh-CN.md)

# bark-notify-skill

Get a push notification on your iPhone the moment Claude Code finishes thinking — so you can step away from the terminal without missing the result.

A Claude Code plugin that uses [Bark](https://github.com/Finb/Bark) to send completion alerts from macOS to your iPhone.

## Why you might want this

- You kicked off a long task and switched to another window — Claude finishes silently and you don't notice for 10 minutes.
- You're away from the desk (kitchen, meeting, etc.) but want to know the moment Claude is ready for your next input.
- You use tmux and want bell-based alerts when Claude Code is interrupted with `Ctrl+C`.

## Requirements

- macOS with Claude Code installed
- iPhone with the [Bark](https://apps.apple.com/app/bark-customed-notifications/id1403753865) app installed and a push URL ready (looks like `https://api.day.app/<your-device-key>`)

## Install

In any Claude Code session, run:

```text
/plugin marketplace add RUIIIOVO/bark-notify-skill
/plugin install bark-notify-skill@bark-notify-skill
/reload-plugins
```

> The first command registers this repo as a plugin marketplace. The second installs the plugin from it. They look similar but are two distinct steps.

## Set it up (1 minute)

After install, run:

```text
/bark-notify-skill:bark-notify-setup
```

You will be asked for:

1. Your Bark push URL — `https://api.day.app/<device-key>`
2. Plain or **encrypted** mode (encrypted is recommended if your push URL is sensitive)
3. If encrypted: a 16-character key (must match your iPhone Bark settings: `AES128` / `CBC` / `pkcs7`)

Then verify it works:

```text
/bark-notify-skill:bark-notify-test
```

You should see a push on your iPhone within a second or two.

## What you'll see

When Claude Code finishes a turn, your iPhone gets a push:

- **Title**: the name of the project directory you're working in
- **Body**: `Claude Code 已完成` ("Claude Code is done")

## Bonus: tmux users

If you run Claude Code inside tmux, enable bell monitoring so you also get notified when Claude is interrupted with `Ctrl+C` (a case where the Stop hook does not fire):

```bash
tmux set -g monitor-bell on
tmux set -g visual-bell on
```

The Bark script emits a terminal bell after every notification, which tmux picks up as pane activity.

## Uninstall

```text
/bark-notify-skill:bark-notify-uninstall
```

This removes the Bark hook from your `~/.claude/settings.json` and deletes the local script.

## Troubleshooting

Just ask Claude in any session — the `bark-notify` skill will guide you through common issues (no notification arriving, encrypted mode mismatch, hook not firing, etc.). Or run setup again to reconfigure.

## Privacy & safety

- Your Bark URL and encryption key stay **on your machine only** — they are written to a local script under `~/.claude/`, never sent anywhere except to the Bark server you control.
- Setup **merges** into your existing `~/.claude/settings.json` rather than overwriting it.
- Encrypted mode means the notification body is AES-encrypted before it leaves your Mac.

---

## For contributors

<details>
<summary>Local development</summary>

```bash
git clone https://github.com/RUIIIOVO/bark-notify-skill.git
cd bark-notify-skill
claude --plugin-dir .
```

Skills included in this plugin:

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `bark-notify` | Auto (model-invoked) | Guidance, setup decisions, troubleshooting |
| `bark-notify-setup` | `/bark-notify-skill:bark-notify-setup` | Interactive setup |
| `bark-notify-test` | `/bark-notify-skill:bark-notify-test` | Sends a verification notification |
| `bark-notify-uninstall` | `/bark-notify-skill:bark-notify-uninstall` | Removes hook and script |

</details>

## License

MIT
