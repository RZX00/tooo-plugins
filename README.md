# tooo-plugins

[中文](#中文) · [English](#english)

## 中文

tooo 是一个待办和计时应用，AI 助手可以直接读写它：添加和修改任务、排日程、开始和暂停计时、补记用时、做日复盘和周复盘、设提醒。到了时间，tooo 在手机、电脑和微信上提醒你。官网 <https://tooo.site>，网页版 <https://app.tooo.site>。

这个仓库把 tooo 打包成插件，装好后你的 AI 助手就连上了 tooo，并带一份说明（技能）告诉它工具怎么用。插件里只有连接地址，没有任何密钥。

**账号目前是邀请制，不开放注册。** 插件只是让你已有的 tooo 账号连到 AI 助手；没有账号的人装了也连不上。

### 安装

连接地址都是 `https://app.tooo.site/mcp`。

**Claude 应用（claude.ai 网页、Windows 和 Mac 桌面端）**

1. 打开 Customize → Plugins → Add → Add marketplace，填 `RZX00/tooo-plugins`。
2. 在 Discover 里选 tooo，点 Add。
3. 打开这个插件的 Connectors 标签，点 Connect，在授权页上同意。装插件本身不会自动连接账号。

**Claude Code**

```
/plugin marketplace add RZX00/tooo-plugins
/plugin install tooo@tooo-plugins
```

第二条命令会打开插件详情，在面板里点安装。装好后输入 `/reload-plugins`，再输入 `/mcp`，选 tooo 完成登录。工具名形如 `mcp__plugin_tooo_tooo__list_tasks`，技能是 `/tooo:tooo`。在终端里也可以直接运行 `claude plugin marketplace add RZX00/tooo-plugins` 和 `claude plugin install tooo@tooo-plugins`，不进会话。

**Codex（CLI 和桌面应用）**

```
codex plugin marketplace add RZX00/tooo-plugins
codex plugin add tooo@tooo-plugins
```

也可以运行 `codex`，输入 `/plugins`，选 tooo 市场，安装 tooo。装好后新开一个会话，按提示完成授权。Codex 的 IDE 扩展不支持插件，请用下面的“只添加连接器”。

### 只添加连接器，不装插件

| 客户端 | 做法 |
| - | - |
| Claude 应用 | 打开 [这个链接](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=tooo&connectorUrl=https%3A%2F%2Fapp.tooo.site%2Fmcp)，表单已填好，点 Add，再点 Connect。也可以手动走 Customize → Connectors → Add custom connector。在网页或桌面端加一次，手机登录同一账号后就能用。 |
| Claude Code | `claude mcp add --transport http --scope user tooo https://app.tooo.site/mcp`，然后在会话里输入 `/mcp` 登录。 |
| Codex | `codex mcp add tooo --url https://app.tooo.site/mcp`，然后 `codex mcp login tooo`。 |
| ChatGPT | 网页版 chatgpt.com/plugins → + → Add custom MCP server，填地址，OAuth 选 CIMD，创建后安装并连接。能否使用取决于你的套餐。 |
| 其他支持 Streamable HTTP 的 Agent | 填同一个地址，授权方式选 OAuth。 |
| Hermes、自己写的 Agent、脚本 | 在 tooo 的“更多 → 设备”里生成命令行密钥（`tk_` 开头），请求头写 `Authorization: Bearer tk_…`。Codex 里可写成 `bearer_token_env_var = "TOOO_KEY"`，把密钥放在环境变量里。 |

### 第一次连接时

授权页由 tooo 账号的所有者批准。浏览器已经登录 tooo 的话，直接点“同意”。没有登录的话，页面会显示 6 位授权码，到已经登录的设备上，在 tooo 应用里打开“更多 → 设备 → 添加设备”，输入授权码批准。批准前，AI 助手看不到你的任何数据。不想用了，在“更多 → 设备”里移除这个连接。

tooo 只把“删除任务”标为每次都要确认的操作。其他写操作是否确认，由各个客户端的设置决定。

### 仓库里有什么

| 路径 | 用途 |
| - | - |
| `.claude-plugin/marketplace.json` | Claude Code 和 Claude 应用读取的市场清单 |
| `.agents/plugins/marketplace.json` | Codex 读取的市场清单 |
| `plugins/tooo/.claude-plugin/plugin.json`、`plugins/tooo/.mcp.json` | Claude 的插件清单和连接配置 |
| `plugins/tooo/plugin.json`、`plugins/tooo/mcp.json` | [Agent Plugins](https://agent-plugins.org) 1.0.0 的插件清单和连接配置，Codex 读取 |
| `plugins/tooo/skills/tooo/SKILL.md` | 两边共用的技能：工具、参数和限制 |

许可证：MIT。

## English

tooo is a to-do list and focus timer that AI assistants can read and write: add and change tasks, plan the day, start and pause the timer, log time, review the day and the week, and set reminders. When it is time, tooo reminds you on your phone, your computer and WeChat. Website: <https://tooo.site>. Web app: <https://app.tooo.site>.

This repository packages tooo as a plugin. Once installed, your AI assistant is connected to tooo and carries a skill that explains the tools. The plugin holds only the connection address, no secrets.

**Accounts are invite-only for now; sign-up is closed.** The plugin connects an account you already have to your AI assistant. Without an account, the connection cannot be approved.

### Install

The connector address is `https://app.tooo.site/mcp` everywhere.

**Claude apps (claude.ai on the web, Windows and Mac desktop)**

1. Open Customize → Plugins → Add → Add marketplace and enter `RZX00/tooo-plugins`.
2. Pick tooo in Discover and click Add.
3. Open the plugin's Connectors tab, click Connect, and approve on the authorization page. Installing the plugin does not connect your account by itself.

**Claude Code**

```
/plugin marketplace add RZX00/tooo-plugins
/plugin install tooo@tooo-plugins
```

The second command opens the plugin's details; install it from that panel. Then run `/reload-plugins` and `/mcp`, and sign in to tooo. Tools are named like `mcp__plugin_tooo_tooo__list_tasks`, and the skill is `/tooo:tooo`. From a shell you can also run `claude plugin marketplace add RZX00/tooo-plugins` and `claude plugin install tooo@tooo-plugins` without opening a session.

**Codex (CLI and desktop app)**

```
codex plugin marketplace add RZX00/tooo-plugins
codex plugin add tooo@tooo-plugins
```

Or run `codex`, type `/plugins`, choose the tooo marketplace and install tooo. Start a new session afterwards and complete the authorization prompt. The Codex IDE extension does not support plugins; use "Connector only" below.

### Connector only, no plugin

| Client | How |
| - | - |
| Claude apps | Open [this link](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=tooo&connectorUrl=https%3A%2F%2Fapp.tooo.site%2Fmcp) (the form is pre-filled), click Add, then Connect. Or go to Customize → Connectors → Add custom connector. Add it once on the web or desktop, and it is available on your phone when you sign in with the same account. |
| Claude Code | `claude mcp add --transport http --scope user tooo https://app.tooo.site/mcp`, then sign in with `/mcp` inside a session. |
| Codex | `codex mcp add tooo --url https://app.tooo.site/mcp`, then `codex mcp login tooo`. |
| ChatGPT | On the web: chatgpt.com/plugins → + → Add custom MCP server, enter the address, choose OAuth with CIMD, create it, install and connect. Availability depends on your plan. |
| Other agents with Streamable HTTP | Use the same address and choose OAuth. |
| Hermes, your own agents, scripts | Generate a command-line key (starts with `tk_`) under More → Devices in tooo and send `Authorization: Bearer tk_…`. In Codex, set `bearer_token_env_var = "TOOO_KEY"` and keep the key in that environment variable. |

### The first connection

The account owner approves the connection on the authorization page. If the browser is already signed in to tooo, click Approve. If not, the page shows a 6-digit code: on a device that is signed in, open More → Devices → Add a device in the tooo app and enter the code. Until you approve, the assistant cannot see any of your data. To disconnect later, remove the connection under More → Devices.

tooo marks only "delete task" as an operation that must be confirmed every time. Whether other write operations ask is up to each client's settings.

### What is in this repository

| Path | Purpose |
| - | - |
| `.claude-plugin/marketplace.json` | Marketplace catalog read by Claude Code and the Claude apps |
| `.agents/plugins/marketplace.json` | Marketplace catalog read by Codex |
| `plugins/tooo/.claude-plugin/plugin.json`, `plugins/tooo/.mcp.json` | Claude plugin manifest and connection config |
| `plugins/tooo/plugin.json`, `plugins/tooo/mcp.json` | [Agent Plugins](https://agent-plugins.org) 1.0.0 manifest and connection config, read by Codex |
| `plugins/tooo/skills/tooo/SKILL.md` | The skill both sides share: tools, parameters and limits |

License: MIT.
