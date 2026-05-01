[English](CHANGELOG.md) | [中文](CHANGELOG.zh-CN.md)

# 更新日志

这个文件记录了此项目的所有重要变更。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)，版本遵循 [Semantic Versioning](https://semver.org/spec/v2.0.0.html)。

## [Unreleased]

## [0.3.0] - 2026-05-02

### Added
- 新增中文更新日志 `CHANGELOG.zh-CN.md`。

### Changed
- 插件包名由 `bark-notify-skill` 更名为 `bark-notify`。
- Marketplace 安装路径调整为 `RUIIIOVO/claude-code-bark-notify`。
- 所有 slash command 示例统一切换为 `bark-notify` 插件命名空间。
- README 与 setup 相关说明统一改为 `bark-notify` 插件名称。

### Notes
- 已安装旧版 `bark-notify-skill` 的用户，需要先卸载旧插件，再重新安装 `bark-notify`。

## [0.2.0] - 2026-05-01

### Added
- 新增 `Notification` hook 支持 —— 当 Claude Code 请求工具调用授权（1/2/3 提示）时会推送通知。
- 按事件类型区分通知正文，让用户一眼就能判断是“该查看”还是“该操作”：
  - `Stop` → `✅ Claude Code 已完成`
  - `Notification` → `🔔 Claude Code 等待操作`
  - `StopFailure` → `❌ Claude Code 出错`
  - `SessionEnd`（异常结束）→ `⚠️ Claude Code 会话异常结束`
- `notification_type` 过滤，并保留基于 `message` 字符串的向后兼容兜底逻辑，兼容旧 payload 结构。

### Changed
- `SessionEnd` 现在从 payload 中读取 `reason`，当会话是用户主动退出（`clear`、`logout`、`prompt_input_exit`）时保持静默。此前每次正常关闭 CLI 都会误发“已完成”推送。
- 四类 hook 事件（`Stop`、`Notification`、`StopFailure`、`SessionEnd`）现在统一走同一个 `~/.claude/claude-stop-bark.sh` 脚本。

### Removed
- 移除自定义 `icon` 字段。由于 `claude.ai/images/...` 链接受 Cloudflare 人机校验保护，Bark 无法成功拉取，最终会显示为坏图标；现在改为使用 Bark 默认图标。
- 移除 `sound` 字段。正文里的 emoji 加上 Bark 的 `level` 已足够区分事件，额外音效属于冗余。

### Fixed
- 修复通知正文始终显示为“Claude Code 已完成”的问题。根因是脚本读取了 Claude Code 实际不会设置的 `$HOOK_EVENT`；事件名真实位于 stdin payload 的 `hook_event_name` 字段。
- 修复 `Notification` hook 同时响应真实授权提示和输入框空闲超时提醒（Claude Code 内置 60 秒 nag）的问题，避免在用户并不需要操作时误发“等待操作”推送。现在已按 `notification_type` 过滤。

## [0.1.0] - 2026-05-01

### Added
- 初始版本发布。
- `Stop`、`StopFailure` 和 `SessionEnd` hooks 触发 `~/.claude/claude-stop-bark.sh`。
- 支持明文模式和 AES-128-CBC 加密的 Bark 推送。
- Skills：`bark-notify`（自动触发引导）、`bark-notify-setup`、`bark-notify-test`、`bark-notify-uninstall`。
- 每次推送后输出终端 bell（`printf '\a'`），用于 tmux pane activity 检测。

[Unreleased]: https://github.com/RUIIIOVO/claude-code-bark-notify/compare/v0.3.0...HEAD
[0.3.0]: https://github.com/RUIIIOVO/claude-code-bark-notify/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/RUIIIOVO/claude-code-bark-notify/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/RUIIIOVO/claude-code-bark-notify/releases/tag/v0.1.0
