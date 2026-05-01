[English](README.md) | [中文](README.zh-CN.md)

# bark-notify-skill

让 Claude Code 跑完任务的瞬间，iPhone 立刻收到推送 — 这样你可以放心离开终端，不用守着屏幕。

一个 Claude Code 插件，通过 [Bark](https://github.com/Finb/Bark) 把 macOS 上的"完成事件"推送到你的 iPhone。

## 你为什么会用到它

- 跑了一个长任务切到别的窗口，Claude 默默跑完了，过了 10 分钟你才发现。
- 人不在桌前（做饭、开会、上厕所），但你想第一时间知道 Claude 是不是已经在等你下一步了。
- 你用 tmux，希望 Claude Code 被 `Ctrl+C` 中断时也能被提醒。

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

> 第一行是把这个仓库注册为插件市场，第二行才是从市场安装插件本体。看起来很像，其实是两个独立步骤。

## 配置（1 分钟）

安装完成后执行：

```text
/bark-notify-skill:bark-notify-setup
```

会依次问你：

1. Bark 推送 URL —— `https://api.day.app/<设备key>`
2. 普通模式 / **加密模式**（推送 URL 比较敏感的话推荐加密）
3. 如果选加密：一个 16 位密钥（需与 iPhone 端 Bark 设置匹配：`AES128` / `CBC` / `pkcs7`）

然后用下面这条验证：

```text
/bark-notify-skill:bark-notify-test
```

正常情况 1~2 秒内 iPhone 上就能看到推送。

## 推送长什么样

每次 Claude Code 完成一轮回答，iPhone 上会收到：

- **标题**：当前项目目录名
- **正文**：`Claude Code 已完成`

## tmux 用户加分项

如果你在 tmux 里跑 Claude Code，建议开启 bell 监听 —— 这样在 `Ctrl+C` 中断的场景（Stop hook 不会触发）也能收到提醒：

```bash
tmux set -g monitor-bell on
tmux set -g visual-bell on
```

Bark 脚本每次推送后都会发一个终端 bell，tmux 检测到 pane 活动就会通知你。

## 卸载

```text
/bark-notify-skill:bark-notify-uninstall
```

会移除 `~/.claude/settings.json` 里的 Bark hook，并删除本地脚本。

## 排错

直接在任意 Claude Code 会话里问就行 —— `bark-notify` skill 会引导你排查常见问题（收不到推送、加密模式对不上、hook 没触发等）。也可以重新跑 setup 覆盖配置。

## 隐私与安全

- 你的 Bark URL 和加密密钥**只存在你本地** —— 写在 `~/.claude/` 下的本地脚本里，除了你自己的 Bark 服务器之外不会发往任何地方。
- Setup 是**合并写入** `~/.claude/settings.json`，不会覆盖你已有的配置。
- 加密模式下，推送正文在离开 Mac 之前就已经被 AES 加密。

---

## 给贡献者

<details>
<summary>本地开发</summary>

```bash
git clone https://github.com/RUIIIOVO/bark-notify-skill.git
cd bark-notify-skill
claude --plugin-dir .
```

插件内包含的 Skills：

| Skill | 触发方式 | 用途 |
|-------|---------|------|
| `bark-notify` | 自动（模型调用） | 引导、决策与排错 |
| `bark-notify-setup` | `/bark-notify-skill:bark-notify-setup` | 交互式配置 |
| `bark-notify-test` | `/bark-notify-skill:bark-notify-test` | 发送验证通知 |
| `bark-notify-uninstall` | `/bark-notify-skill:bark-notify-uninstall` | 卸载 hook 与脚本 |

</details>

## 许可证

MIT
