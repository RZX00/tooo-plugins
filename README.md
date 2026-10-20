[English](./README.en.md)

# tooo-plugins

让 Claude、Codex 直接读写你的 tooo 待办和专注计时：说一句话，任务排好，计时开始，提醒设好。

<p align="center">
  <img src="./docs/assets/banner.jpg" alt="头图：木桌上摊开的手账，左侧标着 6、9、12、15、18、21 点，页面列着一天的待办，其中「Focus time」一行用红色标出并画着番茄；旁边是一杯茶、一个红色番茄计时器和一支笔；封面压印着 tooo" width="100%" />
</p>

tooo 是待办加专注计时应用：任务和步骤、番茄钟、用时记录、日复盘和周复盘、到点提醒。官网 <https://tooo.site>，网页版 <https://app.tooo.site>。本仓库是它的插件：装好后，你的助手连上 tooo，并带着一份技能，说明每个工具怎么用。

## 实际效果

〔待补：证据物，建议内容为：用演示账号，左边是对话「把明天要做的三件事排进 tooo，9 点提醒我」，右边是 tooo 网页里出现的任务。不用真实账号截图，会暴露个人任务。〕

## 状态与限制

- **账号**：tooo 目前内测，账号邀请制，不开放注册。没有账号，装了插件也连不上；助手不会替你注册。
- **能用在哪**：Claude Code、Claude 应用（网页、Windows 和 Mac 桌面端）、Codex（CLI 和桌面应用，IDE 扩展不支持插件）。已实测的是 Claude Code：插件装好后，tooo 的工具和技能都能加载。Claude 应用和 Codex 在 tooo 官网[连接页](https://tooo.site/connect/)标为「已验证」，本仓库没有独立复核〔待核〕。没有本地依赖，不用装 Node 或 Python。
- **数据与权限**：插件里只有连接地址，没有密钥。你批准之前，助手看不到任何数据；批准之后，它能读写这个账号的全部任务、计时、用时记录、周报和提醒设置，读到的内容会进入你所用模型的上下文。删除任务、停止重复、撤销开始计时、去掉已记时间，tooo 标为每次都要确认：Claude Code 每次弹出确认，其他客户端取决于它自己的设置。
- **做不到的**：tooo 自己不调用大模型，总结和建议由你的助手写。它也不会主动唤起助手，所以「每周日自动写周报」要靠你自己的定时任务，或者你在对话里提出。
- **维护**：维护者 tooo，当前版本 0.3.0，没有发布 release；许可证 MIT。

## 安装（三步）

前置条件：一个 tooo 账号；Claude Code、Claude 应用或 Codex 之一。

### 1. 装插件

**推荐**：把这句话发给你的助手，它会按 [`docs/install.md`](./docs/install.md) 装好，并带你授权。

```text
帮我把 tooo 接上：https://raw.githubusercontent.com/RZX00/tooo-plugins/main/docs/install.md
```

自己装，按你用的客户端选一行：

| 客户端 | 做法 |
|---|---|
| Claude Code | `/plugin marketplace add RZX00/tooo-plugins`，再 `/plugin install tooo@tooo-plugins`（第二条会打开插件详情，点安装），装好后输入 `/reload-plugins` |
| Codex | `codex plugin marketplace add RZX00/tooo-plugins`，再 `codex plugin add tooo@tooo-plugins`；也可以在 `codex` 里输入 `/plugins`，选 tooo 市场安装 |
| Claude 应用 | Customize → Plugins → Add → Add marketplace，填 `RZX00/tooo-plugins`；在 Discover 里选 tooo，点 Add |

### 2. 授权（装插件不会自动连上账号）

Claude Code 里输入 `/mcp`，选 tooo 登录；Codex 里新开会话，按提示授权；Claude 应用里打开插件的 Connectors 标签，点 Connect。

授权页由账号所有者批准。浏览器已登录 tooo，直接点「同意」；没有登录，页面会显示 6 位授权码，到已登录的设备上，在 tooo 应用的「更多 → 设备 → 添加设备」里输入。

### 3. 对助手说一句话

见下一节。

## 第一次使用

先读后写。授权完成后，对助手说：

```text
用 tooo，看看我今天有哪些任务
```

你会得到：助手列出今天的任务（用 `list_tasks`，`view` 取 `today`）。这一步只读，用来确认连上了。

再写一条，注意点名 tooo：

```text
用 tooo，把明天要做的三件事排进去：写周报、订机票、去健身。明天早上 9 点提醒我。
```

预期（按 [`SKILL.md`](./plugins/tooo/skills/tooo/SKILL.md) 的工具定义）：明天的列表里多出三条任务，每条在 9:00 有提醒；助手回复里列出标题和提醒时刻。在网页版或手机上能看到同样的三条。〔待补：这一步的真实截图，见「实际效果」〕

再试几句：

| 对助手说 | 用到的工具 |
|---|---|
| `用 tooo，开始专注：写周报` | `start_focus` |
| `用 tooo，补记今天下午两点到三点健身` | `add_time_entry` |
| `用 tooo，看一下我这周的复盘数据` | `get_weekly_review` |
| `用 tooo，把「写周报」改成每周一三五重复` | `set_repeat` |

## 为什么用它

不连接时，助手只能在对话里给你列清单，再由你抄进 tooo；连接后，它直接动 tooo 里的数据。

| 要做的事 | 没有连接 | 装了插件 |
|---|---|---|
| 排明天的事 | 让助手列清单，自己一条条建 | 助手建任务，设好日期、时刻和提醒 |
| 补记用时 | 自己填起止时刻 | 说「下午两点到三点健身」，助手补记，和别的记录重叠的部分记成同时进行 |
| 周复盘 | 自己翻本周记录 | 助手读本周做完的任务、计时和番茄数，起草后用 `save_weekly_review` 存成新一版，旧版保留 |

## 能力与工作原理

助手通过 `https://app.tooo.site/mcp`（Streamable HTTP，OAuth 授权）访问你的账号；技能 `tooo` 告诉它每个工具的参数和限制。[`SKILL.md`](./plugins/tooo/skills/tooo/SKILL.md) 列出 37 个工具，分八类：

| 类别 | 工具 |
|---|---|
| 任务（8） | `list_tasks`、`get_task`、`create_task`、`update_task`、`complete_task`、`delete_task`、`restore_task`、`reorder_tasks` |
| 步骤与重复（4） | `add_step`、`set_repeat`、`stop_repeat`、`list_repeats` |
| 专注计时（10） | `start_focus`、`focus_status`、`pause_focus`、`resume_focus`、`stop_focus`、`cancel_focus`、`start_break`、`end_break`、`get_focus_settings`、`set_focus_settings` |
| 用时记录（2） | `add_time_entry`、`day_timeline` |
| 复盘（4） | `day_summary`、`get_weekly_review`、`list_weekly_reviews`、`save_weekly_review` |
| 提醒（3） | `remind`、`list_reminders`、`cancel_reminder` |
| 主动提醒（5） | `prompts`、`act_prompt`、`cut_prompt_time`、`get_prompt_settings`、`set_prompt_settings` |
| 设备用量（1） | `device_usage` |

### 仓库里有什么

| 路径 | 用途 |
|---|---|
| `.claude-plugin/marketplace.json` | Claude Code 和 Claude 应用读取的市场清单 |
| `.agents/plugins/marketplace.json` | Codex 读取的市场清单 |
| `plugins/tooo/.claude-plugin/plugin.json`、`plugins/tooo/.mcp.json` | Claude 的插件清单和连接配置 |
| `plugins/tooo/plugin.json`、`plugins/tooo/mcp.json` | [Agent Plugins](https://agent-plugins.org) 1.0.0 的插件清单和连接配置，Codex 读取 |
| `plugins/tooo/skills/tooo/SKILL.md` | 两边共用的技能：工具、参数和限制 |
| `docs/install.md` | 给 Agent 读的安装说明 |

## 常见问题

**装了插件，助手说没有 tooo 的工具。** Claude Code 里输入 `/reload-plugins`，或新开会话，再输入 `/mcp` 看 tooo 是否已登录。

**助手能替我点「同意」吗？** 不能。授权页必须由账号所有者本人批准。

**删错了任务能找回吗？** 删除任务和停止重复，都可以让助手用 `restore_task` 撤销。

**不想用了。** 在 tooo 的「更多 → 设备」里移除这个连接。

<details>
<summary>只加连接，不装插件</summary>

只有连接，没有技能那份说明。连接地址都是 `https://app.tooo.site/mcp`。前三条直连做法，tooo 官网[连接页](https://tooo.site/connect/)标为「未验证」（按各家公开文档写的，没有实测）；Hermes Agent 的密钥方式标为「已验证」。

| 客户端 | 做法 |
|---|---|
| Claude 应用 | 打开 [这个链接](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=tooo&connectorUrl=https%3A%2F%2Fapp.tooo.site%2Fmcp)，表单已填好，点 Add，再点 Connect。在网页或桌面端加一次，手机登录同一账号后就能用。 |
| Claude Code | `claude mcp add --transport http --scope user tooo https://app.tooo.site/mcp`，然后在会话里输入 `/mcp` 登录。 |
| Codex | `codex mcp add tooo --url https://app.tooo.site/mcp`，然后 `codex mcp login tooo`。IDE 扩展走这一条。 |
| Hermes Agent | 在 tooo 的「更多 → 设备」里生成命令行密钥（`tk_` 开头），按[连接页](https://tooo.site/connect/)上 Hermes 一栏写进它的配置。密钥不要写进任务或对话。 |

其他客户端也见连接页，那里逐个标了是否验证过。

</details>

## 问题与许可证

问题和建议请提 [Issue](https://github.com/RZX00/tooo-plugins/issues)。许可证 MIT，见 [`LICENSE`](./LICENSE)。
