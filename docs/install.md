# 给 Agent 的安装说明

本文件写给 Agent。用户把本文件的链接发给你，是请你把 tooo 连到你所在的客户端。按下面的顺序做，做完向用户汇报。

## 1 先确认两件事

1. 用户已有 tooo 账号。tooo 目前是内测，账号邀请制，不开放注册。用户没有账号时，如实告诉用户，然后停下；不要替用户注册，也不要试图创建账号。
2. 你要连的地址是 `https://app.tooo.site/mcp`（Streamable HTTP，OAuth 授权）。本仓库里没有任何密钥，也不需要用户给你密钥。

## 2 按你所在的客户端安装

| 你在 | 做法 |
|---|---|
| Claude Code | 在 shell 里运行 `claude plugin marketplace add RZX00/tooo-plugins`，再运行 `claude plugin install tooo@tooo-plugins`。装好后请用户在会话里输入 `/reload-plugins`。 |
| Codex（CLI 或桌面应用） | 在 shell 里运行 `codex plugin marketplace add RZX00/tooo-plugins`，再运行 `codex plugin add tooo@tooo-plugins`。装好后新开一个会话。Codex 的 IDE 扩展不支持插件，改用下面的「只加连接，不装插件」。 |
| Claude 应用（网页、Windows 和 Mac 桌面端） | 你改不了应用设置，把步骤告诉用户：Customize → Plugins → Add → Add marketplace，填 `RZX00/tooo-plugins`；在 Discover 里选 tooo，点 Add。 |
| 其他客户端 | 读 <https://tooo.site/connect/agent.md>，那里按客户端写了做法，并标明了哪些验证过。没验证过的，要告诉用户。 |

只加连接，不装插件（没有技能那份说明，只有连接）：

- Claude Code：`claude mcp add --transport http --scope user tooo https://app.tooo.site/mcp`
- Codex：`codex mcp add tooo --url https://app.tooo.site/mcp`，再运行 `codex mcp login tooo`

## 3 授权

装插件不会自动连上账号，还要授权：

- Claude Code：请用户输入 `/mcp`，选 tooo 登录。
- Codex：新会话里按提示授权。
- Claude 应用：请用户打开插件的 Connectors 标签，点 Connect。

授权页必须由账号所有者本人批准。浏览器已登录 tooo 时，点「同意」即可。没有登录时，页面会显示 6 位授权码，请用户到已经登录的设备上，在 tooo 应用里打开「更多 → 设备 → 添加设备」，输入授权码。你不能替用户点同意，tooo 也没有密码可以输入。

## 4 确认已连上

1. 确认你能看到 tooo 的工具。Claude Code 里工具名形如 `mcp__plugin_tooo_tooo__list_tasks`，技能是 `/tooo:tooo`。看不到时，让用户新开会话，并如实告诉用户，不要说已经装好。
2. 调用 `list_tasks`（`view` 取 `today`），把今天的任务念给用户听，用户由此知道连接可用。这一步只读。

没有授权时，连接地址会返回 401，这是正常的：

```bash
curl -i -X POST https://app.tooo.site/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"probe","version":"0"}}}'
```

2026-10-20 实测返回 `HTTP/1.1 401 Unauthorized`，响应头里有 `WWW-Authenticate: Bearer resource_metadata="https://app.tooo.site/.well-known/oauth-protected-resource/mcp"`，正文是 `{"error":"unauthorized"}`。返回 401 说明地址在线、要求授权；连不上才是网络问题。

## 5 不要做

- 不要替用户注册账号，不要索要密码，不要替用户批准授权。
- 不要把命令行密钥（`tk_` 开头）写进任务、对话或仓库文件。
- 不要为了测试安装而创建、修改或删除用户的任务；确认连接只用上面的只读调用。
- 不要修改 `plugins/tooo/` 下的清单和 `SKILL.md`。

## 6 汇报

装完后告诉用户：装在了哪个客户端、是否已授权、是否确认能看到 tooo 的工具，以及下一步可以对你说的第一句话，例如：

> 用 tooo，看看我今天有哪些任务
