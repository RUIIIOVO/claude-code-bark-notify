# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-05-01

### Added
- `Notification` hook support — pushes when Claude Code asks for tool-call permission (1/2/3 prompt).
- Per-event differentiated notification body so the user can tell at a glance whether to look or to act:
  - `Stop` → `✅ Claude Code 已完成`
  - `Notification` → `🔔 Claude Code 等待操作`
  - `StopFailure` → `❌ Claude Code 出错`
  - `SessionEnd` (abnormal) → `⚠️ Claude Code 会话异常结束`
- `notification_type` filtering with backward-compatible `message`-string fallback for older payload schemas.

### Changed
- `SessionEnd` now reads `reason` from the payload and stays silent when the user initiated the exit (`clear`, `logout`, `prompt_input_exit`). Previously every CLI close fired a stale "completed" push.
- All four hook events (`Stop`, `Notification`, `StopFailure`, `SessionEnd`) now route through a single `~/.claude/claude-stop-bark.sh`.

### Removed
- Custom `icon` field. The `claude.ai/images/...` URL is gated behind Cloudflare human-verification, so Bark could not fetch it. Bark's default icon is used instead.
- `sound` field. Body emoji + Bark's `level` already differentiate events; an audio cue was redundant.

### Fixed
- Notification body was always "Claude Code 已完成" regardless of event. Root cause: the script read `$HOOK_EVENT`, which Claude Code does not set; the event name lives in the stdin payload's `hook_event_name` field.
- `Notification` hook fired for both real permission prompts and idle-input timeouts (Claude Code's built-in 60s nag), causing spurious "等待操作" pushes when no action was actually required. Filtered by `notification_type`.

## [0.1.0] - 2026-05-01

### Added
- Initial release.
- `Stop`, `StopFailure`, and `SessionEnd` hooks fire `~/.claude/claude-stop-bark.sh`.
- Plain mode and AES-128-CBC encrypted Bark transport.
- Skills: `bark-notify` (auto-invoked guidance), `bark-notify-setup`, `bark-notify-test`, `bark-notify-uninstall`.
- Terminal bell (`printf '\a'`) after each notification for tmux pane-activity detection.

[Unreleased]: https://github.com/RUIIIOVO/claude-code-bark-notify/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/RUIIIOVO/claude-code-bark-notify/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/RUIIIOVO/claude-code-bark-notify/releases/tag/v0.1.0
