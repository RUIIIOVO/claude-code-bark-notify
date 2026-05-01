---
name: bark-notify
description: Use this skill whenever the user wants Claude Code on a Mac to notify an iPhone through Bark when a task, turn, or coding session finishes. Trigger on requests about Bark notifications, Claude completion alerts, iPhone push reminders, macOS to iPhone completion messages, Bark hook setup, or Bark troubleshooting, even if the user does not explicitly say skill or plugin.
version: 0.2.0
---

# Bark Notify

This skill helps users set up or troubleshoot Bark notifications for Claude Code completion events on macOS.

## Use this skill for

- guiding a new Bark notification setup
- deciding between plain and encrypted Bark push
- troubleshooting Bark failures, decryption failures, or missing notifications
- explaining how Claude Code `Stop` hooks work for iPhone notifications

## Working style

Prefer the lightest path that matches the user's intent.

- If the user wants explanation only, guide them.
- If the user wants direct setup, route them to `/bark-notify:bark-notify-setup`.
- If the user wants to verify delivery, route them to `/bark-notify:bark-notify-test`.

## Scope

This plugin focuses on one concrete outcome:

- macOS Claude Code session
- Bark notification on iPhone
- notification sent when Claude finishes a turn

The default notification should stay minimal for readability:
- title = current project name
- body = `Claude Code 已完成`

## References

Read these only when relevant:

- `references/bark-prereqs.md` for Bark prerequisites and required user inputs
- `references/config-patterns.md` for plain vs encrypted hook patterns
- `references/troubleshooting.md` for failed delivery or decryption issues
