[English](README.md) | [中文](README.zh-CN.md)

# bark-notify

一个 Claude Code 插件，通过 [Bark](https://github.com/Finb/Bark) 把 macOS 上的完成、出错、等待授权等事件推送到 iPhone。

## 我为什么做这个插件

- `vibe coding`，尤其是多个项目并行的时候，很容易隔一会儿就需要回头看一眼终端：Claude 到底跑完了没有，是出错了，还是卡在等我确认。
- 在公司里不方便开声音提醒；一边看手机、一边并行做别的事，又总会被“要不要回终端看一眼”这件事反复打断，很烦。
- Claude 官方也有移动端消息通知的功能，但更偏向 `Cowork` / `Dispatch` / `Remote Control` 这套体验：要装 Claude 手机 App、打开通知、保持连接可用，在中国国内必须使用魔法不然容易封号，同时开着魔法摸鱼带来的网络延迟也会让人很不爽。
- 如果你主要用的是 `Claude Code CLI`，官方给到的可定制能力目前主要还是 `hooks`；官方的手机推送能力更多在 `Remote Control` / `Dispatch` 这条链路上，不是纯 CLI 默认自带的一条轻量通知路径。
- 市面上当然也有别的办法，比如接飞书、钉钉机器人，或者走 `iMessage`、邮箱，但不是配置重、权限多，就是过于依赖特定平台，不太适合这种高频提醒。
- 另外，如果你平时会用 `ccswitch` 切模型，官方桌面端那条路也不一定顺手。我自己现在是 `ccswitch` + 豆包语音输入法 + 这个插件，`vibe coding` 的体验会顺很多。
- 所以我做了这个插件：用 Bark 把“该提醒我的时候提醒我”这件事单独做好。它配置简单，App 装好后不用额外注册，自动生成 key，把 URL 交给 Claude 就能很快配好。
- 如果你在意隐私，也可以直接开加密，或者把 Bark 部署到自己的服务器上；至少在“提醒”这一层，不需要再额外绕一遍企业 IM 或别的第三方通知链路。

## 前置条件

- macOS 装好 Claude Code
- iPhone 装好 [Bark](https://apps.apple.com/cn/app/bark-customed-notifications/id1403753865)，并拿到推送 URL（形如 `https://api.day.app/<你的设备key>`）

## 安装

在任意 Claude Code 会话中执行：

```text
/plugin marketplace add RUIIIOVO/claude-code-bark-notify
/plugin install bark-notify@bark-notify
/reload-plugins
```

## 配置

```text
/bark-notify:bark-notify-setup
```

会依次问你：

1. Bark 推送 URL —— `https://api.day.app/<设备key>`
2. 普通模式 / **加密模式**（推送 URL 比较敏感的话推荐加密）
3. 如果选加密：一个 16 位密钥（需与 iPhone 端 Bark 设置匹配：`AES128` / `CBC` / `pkcs7`）

验证：

```text
/bark-notify:bark-notify-test
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
/bark-notify:bark-notify-uninstall
```

会移除 `~/.claude/settings.json` 里的 Bark hook，并删除本地脚本。

## 排错

在任意 Claude Code 会话里直接问 —— `bark-notify` 插件内置了引导用的 skill，也可以重新跑 setup 覆盖配置。

## 隐私

- Bark URL 和加密密钥仅存在 `~/.claude/` 本地脚本里，除你自己的 Bark 服务器外不会发往任何地方。
- Setup 是合并写入 `~/.claude/settings.json`，不会覆盖你已有的配置。
- 加密模式下，推送正文在离开 Mac 之前已经过 AES-128-CBC 加密。

---

## 给贡献者

<details>
<summary>本地开发</summary>

```bash
git clone https://github.com/RUIIIOVO/claude-code-bark-notify.git
cd claude-code-bark-notify
claude --plugin-dir .
```

Skills：

| Skill | 触发方式 | 用途 |
|-------|---------|------|
| `bark-notify` | 自动（模型调用） | 引导、决策与排错 |
| `bark-notify-setup` | `/bark-notify:bark-notify-setup` | 交互式配置 |
| `bark-notify-test` | `/bark-notify:bark-notify-test` | 发送验证通知 |
| `bark-notify-uninstall` | `/bark-notify:bark-notify-uninstall` | 卸载 hook 与脚本 |

</details>

## 更新日志

见 [CHANGELOG.md](CHANGELOG.md)。

## 许可证

MIT
