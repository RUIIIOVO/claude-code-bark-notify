[English](README.md) | [中文](README.zh-CN.md)

# bark-notify-skill

Claude Code 的 Bark 推送通知插件。当 Claude Code 在 macOS 上完成一轮对话时，自动通过 Bark 向 iPhone 发送通知。

## 工作原理

- iPhone 上安装 Bark 应用
- macOS 上运行 Claude Code
- 通过 Claude Code 的 `Stop` 钩子触发本地脚本
- 脚本调用 Bark API 发送推送通知

## 功能说明

| Skill | 触发方式 | 用途 |
|-------|---------|------|
| `bark-notify` | 自动触发（模型调用） | 提供引导、决策和故障排查 |
| `bark-notify-setup` | `/bark-notify-skill:bark-notify-setup` | 交互式配置，写入本地脚本和钩子 |
| `bark-notify-test` | `/bark-notify-skill:bark-notify-test` | 发送测试通知，验证配置 |

## 通知格式

- **标题**：当前项目名称
- **正文**：`Claude Code 已完成`

## 安装方式

在 Claude Code 会话内执行：

```bash
/plugin marketplace add RUIIIOVO/bark-notify-skill
/plugin install bark-notify-skill@bark-notify-skill
```

执行 `/reload-plugins` 激活。

## 快速开始

安装后执行：

```
/bark-notify-skill:bark-notify-setup
```

按提示完成：
1. 输入 Bark 推送 URL（格式：`https://api.day.app/<设备key>`）
2. 选择推送模式（普通 / 加密）
3. 加密模式需提供 16 位密钥

配置完成后执行 `/bark-notify-skill:bark-notify-test` 验证。

## 加密模式

需要：
- Bark 推送 URL
- 16 位加密密钥
- iPhone Bark 应用对应设置：`AES128` + `CBC` + `pkcs7`

## 配置会修改的文件

Setup 流程确认后写入两个文件：
- `~/.claude/settings.json` — 添加 `Stop` 钩子（合并写入，不覆盖已有配置）
- `~/.claude/claude-stop-bark.sh` — Bark 推送脚本

## 仓库结构

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

## 本地开发

```bash
git clone https://github.com/RUIIIOVO/bark-notify-skill.git
cd bark-notify-skill
claude --plugin-dir .
```

## 许可证

MIT
