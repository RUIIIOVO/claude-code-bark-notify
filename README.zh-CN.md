[English](README.md) | [中文](README.zh-CN.md)

# bark-notify-skill

一个 Claude Code 插件，通过 [Bark](https://github.com/Finb/Bark) 把 macOS 上的完成、出错、等待授权等事件推送到 iPhone。

## 你为什么会想用

- Claude 30 秒就跑完了，你 12 分钟后才回过神。
- 你出去倒杯水回来，发现 Claude 在等你点确认已经站岗 5 分钟。
- 你的终端在 tmux 里，手机在另一个房间——推送照样能到。
- 合上电脑离开桌子。推送照样到。

## 前置条件

- macOS 装好 Claude Code
- iPhone 装好 [Bark](https://apps.apple.com/cn/app/bark-customed-notifications/id1403753865)，并拿到推送 URL（形如 `https://api.day.app/<你的设备key>`）

## 安装

在任意 Claude Code 会话中执行：

```text
/plugin marketplace add RUIIIOVO/bark-notify-skill
/plugin install bark-notify-skill@bark-notify-skill
/reload-plugins
```

## 配置

```text
/bark-notify-skill:bark-notify-setup
```

会依次问你：

1. Bark 推送 URL —— `https://api.day.app/<设备key>`
2. 普通模式 / **加密模式**（推送 URL 比较敏感的话推荐加密）
3. 如果选加密：一个 16 位密钥（需与 iPhone 端 Bark 设置匹配：`AES128` / `CBC` / `pkcs7`）

验证：

```text
/bark-notify-skill:bark-notify-test
```

## 推送内容

推送**标题**始终是当前项目目录名。**正文**按事件类型区分：

| 触发场景 | 正文 | Bark `level` |
|----------|------|--------------|
| Claude 一轮回答正常结束 | `✅ Claude Code 已完成` | `active` |
| Claude 需要你授权工具调用（出现 1/2/3 选项） | `🔔 Claude Code 等待操作` | `timeSensitive` |
| 一轮回答因错误中断 | `❌ Claude Code 出错` | `timeSensitive` |
| 会话异常结束（崩溃、被杀） | `⚠️ Claude Code 会话异常结束` | `timeSensitive` |
| 你主动关闭 CLI（`/exit` / `/clear` / `/logout`） | **静默** | — |
| Claude Code 输入框空闲 60s 的内置提醒 | **静默** | — |

## tmux

脚本每次推送后都会发一个终端 bell，要让 tmux 接收：

```bash
tmux set -g monitor-bell on
tmux set -g visual-bell on
```

## 卸载

```text
/bark-notify-skill:bark-notify-uninstall
```

会移除 `~/.claude/settings.json` 里的 Bark hook，并删除本地脚本。

## 排错

在任意 Claude Code 会话里直接问 —— `bark-notify` skill 会引导你。也可以重新跑 setup 覆盖配置。

## 隐私

- Bark URL 和加密密钥仅存在 `~/.claude/` 本地脚本里，除你自己的 Bark 服务器外不会发往任何地方。
- Setup 是合并写入 `~/.claude/settings.json`，不会覆盖你已有的配置。
- 加密模式下，推送正文在离开 Mac 之前已经过 AES-128-CBC 加密。

---

## 给贡献者

<details>
<summary>本地开发</summary>

```bash
git clone https://github.com/RUIIIOVO/bark-notify-skill.git
cd bark-notify-skill
claude --plugin-dir .
```

Skills：

| Skill | 触发方式 | 用途 |
|-------|---------|------|
| `bark-notify` | 自动（模型调用） | 引导、决策与排错 |
| `bark-notify-setup` | `/bark-notify-skill:bark-notify-setup` | 交互式配置 |
| `bark-notify-test` | `/bark-notify-skill:bark-notify-test` | 发送验证通知 |
| `bark-notify-uninstall` | `/bark-notify-skill:bark-notify-uninstall` | 卸载 hook 与脚本 |

</details>

## 更新日志

见 [CHANGELOG.md](CHANGELOG.md)。

## 许可证

MIT
