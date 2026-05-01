[English](README.md) | [中文](README.zh-CN.md)

# bark-notify-skill

A Claude Code plugin that sends a Bark push notification to your iPhone when Claude finishes a turn on macOS.

## How it works

- Bark app installed on iPhone
- Claude Code running on macOS
- A `Stop` hook triggers a local shell script
- The script calls the Bark API to send a push notification

## Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `bark-notify` | Auto (model-invoked) | Guidance, setup decisions, and troubleshooting |
| `bark-notify-setup` | `/bark-notify-skill:bark-notify-setup` | Interactive setup — writes local script and hook |
| `bark-notify-test` | `/bark-notify-skill:bark-notify-test` | Sends a verification notification |

## Notification format

- **Title**: current project name
- **Body**: `Claude Code 已完成`

## Installation

Run these two commands in a Claude Code session:

```bash
/plugin marketplace add RUIIIOVO/bark-notify-skill
/plugin install bark-notify-skill@bark-notify-skill
```

Then `/reload-plugins` to activate.

## Quick start

After installation, run:

```
/bark-notify-skill:bark-notify-setup
```

Follow the prompts:
1. Enter your Bark push URL (`https://api.day.app/<device-key>`)
2. Choose plain or encrypted mode
3. If encrypted, provide a 16-character encryption key

Then verify with `/bark-notify-skill:bark-notify-test`.

## Encrypted mode

Requires:
- Bark push URL
- 16-character encryption key
- Matching Bark iPhone settings: `AES128` + `CBC` + `pkcs7`

## What setup modifies

The setup flow writes two local files (after confirmation):
- `~/.claude/settings.json` — adds a `Stop` hook (merged, never overwrites existing config)
- `~/.claude/claude-stop-bark.sh` — the Bark push script

## Repository structure

```text
bark-notify-skill/
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── skills/
│   ├── bark-notify/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── bark-prereqs.md
│   │       ├── config-patterns.md
│   │       └── troubleshooting.md
│   ├── bark-notify-setup/
│   │   └── SKILL.md
│   └── bark-notify-test/
│       └── SKILL.md
├── README.md
├── README.zh-CN.md
├── LICENSE
└── .gitignore
```

## Local development

```bash
git clone https://github.com/RUIIIOVO/bark-notify-skill.git
cd bark-notify-skill
claude --plugin-dir .
```

## License

MIT
