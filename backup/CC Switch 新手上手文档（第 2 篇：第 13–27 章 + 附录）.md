> **接第 1 篇：** 第 0–12 章见另一篇 Issue（概念、安装、供应商、切换、验证、本地路由、故障转移、聚合）。本篇从第 13 章开始。

## 13. Claude Desktop 专项

### 13.1 它和 Claude Code 的区别

| | Claude Code | Claude Desktop |
| :-- | :-- | :-- |
| 配置文件 | `~/.claude/settings.json` | 3P profile 配置 |
| 生效方式 | 热重载 | **必须重启应用** |
| MCP / Skills / 提示词 / 会话 | 有 | **没有**（面板操作的是 Claude Code 的数据） |
| 连接模式 | 直连 / 路由 / 聚合 | 直连 / 模型映射 |

### 13.2 配置文件位置

| 系统 | 位置 |
| :-- | :-- |
| macOS | `~/Library/Application Support/Claude/` 和 `Claude-3p/` |
| Windows | `%LOCALAPPDATA%` 下的 `Claude` 和 `Claude-3p` |
| Linux | `~/.config/Claude/` 和 `Claude-3p/` |

```
Claude/claude_desktop_config.json        # 主配置
Claude-3p/claude_desktop_config.json     # 第三方推理（gateway）模式的配置
Claude-3p/configLibrary/*.json           # CC Switch 写入的 gateway 配置
```

### 13.3 从 Claude Code 一键导入

第一次进入 Claude Desktop 页时，如果这里还没有供应商，可以直接点 **「将 Claude Code 中已有的供应商导入」**。

**导入规则：**

- 已存在同 ID 供应商时不会覆盖；
- 模型名为三档角色 ID（`claude-sonnet-*` / `claude-opus-*` / `claude-haiku-*`）且能直连的 → 导入为**直连模式**；
- 模型名非三档角色 ID（含旧式 Claude ID）或需要格式转换的 → 尝试导入为**模型映射模式**；
- `ANTHROPIC_DEFAULT_SONNET_MODEL` 等会转换为 Desktop 的 Sonnet / Opus / Haiku 映射；
- 旧的 `[1M]` 后缀会转换为 `supports1m` 标记；
- 无法判断模型映射的供应商会被跳过。

> 导入后请检查每个供应商的模型映射是否符合你的实际上游模型。

### 13.4 两种接入方式

#### 直连

适合供应商本身提供 **Anthropic Messages API**，且模型名是 Claude Desktop 可识别的三档角色 ID。

CC Switch 会把 3P profile 指向供应商接口：

```json
{
  "inferenceProvider": "gateway",
  "inferenceGatewayBaseUrl": "https://api.example.com",
  "inferenceGatewayAuthScheme": "bearer",
  "inferenceGatewayApiKey": "你的 API Key"
}
```

**适用场景：**

- 供应商暴露原生 Anthropic Messages API；
- 模型 ID 为 `claude-sonnet-*`、`claude-opus-*`、`claude-haiku-*`；
- 不需要格式转换；
- **不需要 CC Switch 在使用期间保持运行**。

> 「手动指定 Claude Desktop 模型列表」是高级选项，多数原生 Claude 模型供应商不需要填写（Desktop 会自动读 `/v1/models`）。手动填的模型名必须是三档角色 ID 形态，旧式 `claude-3-5-sonnet-…` 会被拒绝。

#### 模型映射

适合供应商的模型**不是**三档角色 ID（包括旧式 Claude ID 和 DeepSeek、Kimi 等非 Claude 模型），或接口格式需要转换。

开启「需要模型映射」后，Claude Desktop 会连接到 CC Switch 本地网关：

```
http://127.0.0.1:15721/claude-desktop
```

CC Switch 会负责：

1. 向 Claude Desktop 暴露安全的 Claude 模型路由；
2. 把 Desktop 选择的模型角色映射到真实上游模型；
3. 按供应商要求转换 Anthropic / OpenAI / Gemini 请求格式；
4. 用 CC Switch 中保存的供应商凭据访问上游。

> **模型映射模式需要 CC Switch 保持运行。**

### 13.5 切换与重启

1. 在供应商卡片上点 **「切换」**；
2. **完全退出并重新打开 Claude Desktop**；
3. 使用模型映射时，保持 CC Switch 运行。

---

## 14. Claude 官方多账号

> 适用版本：**CC Switch v4.0.21 及以上**。

### 14.1 能做什么

很多人同时有好几个 Claude 订阅（公司账号 + 个人账号）。Claude Code 同一时间只认一个登录，换号只能 `/logout` 再 `/login`。CC Switch 提供两种用法：

| 用法 | 说明 |
| :-- | :-- |
| **切卡即切号** | 在 Claude 官方卡上选一个账号，切到这张卡，Claude Code 的登录就实时换成这个号：正在运行的会话不用重启，接下来的请求走新账号；新开的终端、IDE 插件、脚本里的 `claude` 也会用新的 |
| **一个窗口一个号** | 从卡片「打开终端」，这个终端固定用卡片上的账号，环境隔离，几个号可以同时开启；也适用于第三方供应商 |

每张绑了账号的卡片还会显示这个号的**套餐和剩余额度**。

> ⚠️ **只支持 Claude Code CLI 和 IDE 插件（VS Code、JetBrains 等），不支持 Claude Desktop，不支持 WSL 环境。**
>
> Claude Desktop 的聊天和 Code 标签页用的是它自己的登录，不读 Claude Code 的凭据。

### 14.2 用法

**第 1 步：添加账号**

1. 打开左侧栏 **授权中心**；
2. 在 Claude 组点 **「添加账号」**；
3. 在弹出的浏览器里用 claude.ai 登录；
4. 几个号就登录几次。

> 浏览器授权后没有回到 CC Switch 时，点「授权后没有回到这里？」，把浏览器地址栏里的地址或页面上的代码粘贴进去。
>
> Claude Code 里现在登着的号不用专门添加，**第一次切换时会自动收录**。

**第 2 步：给官方卡选账号**

1. 在 Claude Code 页面添加或编辑一张 `Claude Official` 卡；
2. 在 **「登录方式」** 下拉里选一个账号；
3. 建议**一个号一张卡**，卡名写用途（如「Claude 工作」「Claude 个人」）。

> 下拉最下面的 **「跟随 Claude Code 登录」** 表示不绑号，Claude Code 现在登着谁就用谁。

**第 3 步：即时切换**

点卡片的 **「启用」**，Claude Code 的登录就实时换成这个号。

**注意：**

- 正在运行的会话也会跟着换号，所以切号后正在使用的 Remote Control 会断开，下一条消息要重建提示词缓存（会慢一点、多用一点额度）；
- 仅**直连模式**；
- 在设置里覆盖了 Claude 配置目录时，切卡不会换号，请用「打开终端」；
- WSL 环境下两种用法都不支持。

### 14.3 合规说明（重要）

Anthropic 在 [Claude Code 法律与合规页](https://code.claude.com/docs/en/legal-and-compliance#authentication-and-credential-use) 的「Authentication and credential use」一节要求：

- 订阅登录只给订阅者本人，用于正常使用 Claude Code；
- 第三方不能把 Claude 登录做进自己的应用，也不能拿用户的订阅凭据替用户转发请求；
- 第三方不能收集、存储或中转会话令牌；
- 这些限制不妨碍用户用自己的订阅登录未经修改的 Claude Code。

**CC Switch 的对照：**

| 官方要求 | CC Switch 的做法 |
| :-- | :-- |
| 登录走 Anthropic 自己的流程 | 添加账号时运行 Claude Code 官方的 `claude auth login`，登录和授权都在 claude.ai 官方页面完成 |
| 不把 Claude 登录做进自己的应用 | CC Switch 没有自己的 Claude 登录和 OAuth 客户端；每个账号就是一个官方推荐的 `CLAUDE_CONFIG_DIR` 目录 |
| 不替用户转发请求 | 不拿订阅令牌发模型请求，也没有服务器；对话请求由本机未经修改的 Claude Code 直接发给 Anthropic |
| 不收集、存储、中转令牌 | 令牌始终以 Claude Code 自己的格式保存在本机；CC Switch 的数据库、配置文件和云同步里都没有令牌 |
| 只供本人正常使用 | 都是你自己的订阅，各用各的额度；CC Switch 不做按额度自动轮换 |

此外，CC Switch **不刷新令牌**、**不登出、不吊销**、**不修改 Claude Code 程序本身**。

> **「切卡即切号」的边界**：为了让 IDE 插件和脚本也能换号，切卡时会在本机 Claude Code 的存储位置之间移动登录，令牌不离开本机。官方文档没有专门讲到这种用法。
>
> 如果只想用官方写明的方式，可以给账号建卡但不「启用」，只用卡片上的「打开终端」。

---

## 15. 授权中心与订阅当供应商

### 15.1 授权中心是什么

左侧栏 **「授权中心」**（Beta），统一管理第三方 OAuth 凭据：

| 账号类型 | 用途 |
| :-- | :-- |
| **GitHub Copilot** | 配合 Copilot 反向代理使用 |
| **ChatGPT（Codex OAuth）** | 配合 Codex OAuth 反向代理使用，管理 ChatGPT 账号 |
| **xAI（Grok OAuth）** | 配合 xAI Responses API 反向代理使用 |
| **Kimi Code** | 同时保存多个官方账号，每个账号使用独立凭据和原生供应商 |
| **Claude** | 官方多账号（见 [14](#14-claude-官方多账号)） |

**能做什么：**

- 通过 Device Code 流程登录；
- 查看已登录账号列表和认证状态；
- 为多账号设置默认账号；
- 移除单个账号或一键注销所有账号。

### 15.2 Codex OAuth：在 Claude Code 里用 ChatGPT 订阅

**前提条件：**

- 可登录的 **ChatGPT 账号**；
- 能访问 `auth.openai.com` 和 `chatgpt.com`。

**两个入口：**

**入口 A（推荐新用户）：**

1. 切到 **Claude** 应用；
2. 点页头 **添加供应商**；
3. 在「账号登录」分类里选 **Codex（用 ChatGPT 账号登录）** 预设；
4. 未登录时表单会引导你进入登录流程；
5. 登录成功后，表单显示已登录账号，点「添加」。

**入口 B（适合多账号管理）：**

1. 点侧栏 **授权中心**；
2. 在 **ChatGPT** 区块点 **添加账号**；
3. 完成登录；
4. 回到 Claude → 添加供应商 → 选同一个 Codex 预设；
5. 在「选择账号」下拉里选刚登录的账号。

**登录流程（Device Code）：**

1. CC Switch 调用 OpenAI Device Code 流程，显示一个**验证码**（约 8 位，如 `ABCD-1234`）和授权链接 `https://auth.openai.com/codex/device`；
2. 点击链接（或手动访问），在浏览器里登录 ChatGPT 账号、输入验证码、确认授权；
3. CC Switch 后台持续轮询，检测到授权成功后自动关闭等待界面；
4. 登录的账号出现在**授权中心**的 ChatGPT 区块。

> ⏱️ **验证码有效期约 15 分钟。** 超时后重新发起登录即可。

**启用：**

1. 卡片上显示「需要路由」徽章；
2. 切到「路由」标签，点「开始路由」；
3. 点卡片上的「路由到此」。

> **底层细节**：请求会被路由到 `https://chatgpt.com/backend-api/codex`，Base URL 被强制重写——**无需手动填端点**。API 格式固定为 `openai_responses`。

**多账号管理：**

| 操作 | 说明 |
| :-- | :-- |
| 添加其他账号 | 点「添加账号」重复登录流程 |
| 设为默认 | 在账号行的更多操作里选「设为默认」 |
| 为供应商选账号 | 供应商表单中通过「选择账号」下拉框指定 |
| 重新登录 | 账号显示「需要重新登录」时，在更多操作里选「重新登录」 |
| 移除账号 | 在账号行的更多操作里选「删除账号」 |

**Token 自动刷新：**

- 在**过期前 60 秒**自动刷新，全程后台；
- Refresh Token 存储在本地数据目录，不会上传；
- **不支持**导出 Token。

**常见失败：**

| 场景 | 表现 | 解决 |
| :-- | :-- | :-- |
| 验证码超时 | 提示登录没有完成 | 重新发起登录 |
| 浏览器拒绝授权 | 提示登录没有完成 | 重新登录并点「授权」 |
| 网络错误 | 显示具体错误 | 检查网络和代理 |
| 添加供应商前未登录 | 「请先登录 ChatGPT 账号」 | 先到授权中心登录 |
| Token 失效无法刷新 | 账号行显示「需要重新登录」 | 对该账号重新登录 |

### 15.3 GitHub Copilot

通过授权中心登录 GitHub 账号后，可在 **Claude Code、Claude Desktop 和 Codex** 中添加 GitHub Copilot 供应商。

**上游协议：**

- **Claude Code / Claude Desktop**：上游协议由路由**按模型自动选择**，手选格式不生效（表单会禁用格式选择并显示说明）；
- **Codex**：可选择 `Auto`、`Chat Completions` 或 `Responses`，模型列表按所选协议过滤；
- 普通供应商仍按手选格式转发。

> Copilot 的额度会**自动显示**，无需配置用量查询。

### 15.4 xAI（Grok）

使用 OAuth 2.0 Device Authorization Grant，**不使用本机回环回调**：

| 依赖 | 端点 |
| :-- | :-- |
| OpenID discovery | `https://auth.x.ai/.well-known/openid-configuration` |
| 设备授权 | discovery 返回的 `device_authorization_endpoint` |
| Token 交换/刷新 | discovery 返回的 `token_endpoint` |
| 推理 | `https://api.x.ai/v1/responses` |
| 模型发现 | `https://api.x.ai/v1/models` |

流程不占用任何本地回调端口；认证和模型请求都复用 CC Switch 的全局出站代理设置。

### 15.5 Kimi Code

- 在授权中心 **Kimi Code** 分组里选择**国内版或海外版**，点「登录 Kimi」或「添加账号」；
- 在浏览器里输入授权码并确认；
- 支持同时登录多个官方账号；
- 每个账号使用独立凭据和原生供应商；
- 首个登录沿用 Kimi 原生默认凭据槽位。

| 登录区域 | 授权主机 | 模型 API |
| :-- | :-- | :-- |
| 国内版 | `https://auth.kimi.com` | `https://api.kimi.com/coding/v1` |
| 海外版 | `https://auth.kimi.ai` | `https://api.kimi.ai/coding/v1` |

> 仍在原生配置中的 Kimi 官方 OAuth 节点在供应商页受到保护；原生节点被移除后，可以删除留下的档案。

### 15.6 ⚠️ 风险提示（必读）

**以下功能通过逆向工程的 OAuth 流程访问厂商服务：**

- Codex OAuth 反向代理（在 Claude Code 里用 ChatGPT 订阅）
- GitHub Copilot 反向代理
- xAI OAuth 反向代理

**风险：**

1. **可能违反服务条款**：条款通常禁止未经授权的自动化访问、服务复制和绕过既定访问路径；
2. **账号风险**：厂商可能将异常使用模式标记为可疑自动化，对账号施加临时或永久限制；
3. **无法保证长期可用**：厂商随时可能更新认证和检测机制，当前可用的方式未来可能被封堵；
4. **复用上游产品的 OAuth 客户端可能不受官方支持**（xAI 集成复用 Grok CLI 注册的公共 OAuth 客户端身份）。

> **启用这些功能即表示你自行承担所有风险。** CC Switch 不对因使用本功能产生的账号限制、警告或服务暂停承担责任。
>
> 使用第三方中转站时，你的 prompt 和 Key 都会经过中转站——只选你信任的。

---

## 16. MCP、提示词、Skills、会话、工作区

### 16.1 MCP 服务器管理

**打开**：左侧栏 **MCP**（全局页，不属于某个应用）。

**界面**：一张**矩阵表**——每行一个 MCP 服务器，每列一个应用，格子里的开关就是「这个 MCP 在这个应用里启用」。

**页头：**

- **从应用导入**：读取各工具现有的 MCP 配置；
- **添加 MCP**；
- **⋯ 菜单**：「按这里的开关重写各应用配置」「复制全部为 mcpServers JSON」。

**添加方式：**

| 方式 | 说明 |
| :-- | :-- |
| **从模板开始** | fetch、time、memory、sequential-thinking、context7 |
| **粘贴配置** | 直接贴 JSON 或 TOML，自动识别 `mcpServers`、Codex 的 `[mcp_servers.名称]`、OpenCode 的 local / remote 写法 |
| **手动填写** | 名称、连接方式、命令、参数、环境变量、工作目录、地址、请求头 |

**常用模板：**

| 模板 | 包名 | 功能 |
| :-- | :-- | :-- |
| fetch | `mcp-server-fetch` | HTTP 请求，让 AI 能获取网页内容 |
| time | `mcp-server-time`（通过 `uvx`） | 当前时间 |
| memory | `@modelcontextprotocol/server-memory` | 存储和检索信息 |
| sequential-thinking | `@modelcontextprotocol/server-sequential-thinking` | 增强推理 |
| context7 | `@upstash/context7-mcp` | 查询技术文档 |

**传输类型：**

| 类型 | 说明 | 示例 |
| :-- | :-- | :-- |
| **stdio** | 启动本地进程通信（最常用） | `{"command":"uvx","args":["mcp-server-fetch"]}` |
| **http** | 远程 HTTP | `{"url":"http://localhost:8080/mcp"}` |
| **sse** | Server-Sent Events | `{"url":"http://localhost:8080/sse"}` |

> **Pi 不支持 SSE**；Pi 的服务器 ID 只能包含字母、数字、`_` 和 `-`，且只差 `-` 和 `_` 的两个 ID 算同一个。

**支持 MCP 同步的应用及写入位置：**

| 应用 | 配置文件 |
| :-- | :-- |
| Claude Code | `~/.claude.json` 的 `mcpServers` |
| Codex | `~/.codex/config.toml` 的 `[mcp_servers]` |
| Gemini CLI | `~/.gemini/settings.json` 的 `mcpServers` |
| Grok Build | `~/.grok/config.toml` 的 `[mcp_servers]` |
| OpenCode | `~/.config/opencode/opencode.json` 的 `mcp` |
| Hermes | `~/.hermes/config.yaml` 的 `mcp_servers` |
| Pi | `~/.pi/agent/mcp.json`（需 Pi 1.0+） |
| MiniMax Code | `~/.minimax/mcp.json` |
| Kimi Code | `~/.kimi-code/mcp.json` |
| Oh My Pi | `~/.omp/agent/mcp.json` |

> 用户手册 3.1 仍写「8 个应用」，README 已把 **Kimi Code、Oh My Pi** 列入统一 MCP 面板。以应用内实际列出的列为准。
>
> **Claude Desktop 和 OpenClaw 不支持** MCP 同步。

**同步条件**：对应应用已安装（目录存在）。未安装时开关不报错，但配置不会写入。

**连通性检查**（v4.0.25 新增）：MCP 页有「检查连通」按钮，每行菜单有「测试连接」；单个服务器 30 秒超时，全局最多 3 个并发。

### 16.2 提示词（Prompts）

**打开**：左侧栏 **提示词**（全局页）。页头下拉框选应用。

**支持的应用**：Claude Code、Codex、Gemini CLI、Grok Build、OpenCode、Hermes、Pi、MiniMax Code、Kimi Code。

> 用户手册 3.2 仍列 8 个应用；README 已把 **Kimi Code** 列入统一提示词页（写入 `~/.kimi-code/AGENTS.md`）。Oh My Pi 在 README 的功能表里也标了 `AGENTS.md`。

> Claude Desktop 没有自己的提示词，从它页面进入时操作的是 Claude Code 的；OpenClaw 用「工作区编辑器」管理；Hermes 在这里管理的是人设文件 `SOUL.md`。

**同步目标：**

| 应用 | 文件 |
| :-- | :-- |
| Claude Code | `~/.claude/CLAUDE.md` |
| Codex | `~/.codex/AGENTS.md` |
| Gemini CLI | `~/.gemini/GEMINI.md` |
| Grok Build | `~/.grok/AGENTS.md` |
| OpenCode | `~/.config/opencode/AGENTS.md` |
| Hermes | `~/.hermes/SOUL.md` |
| Pi | `~/.pi/agent/AGENTS.md` |
| MiniMax Code | `~/.minimax/AGENTS.md`（不超过 32 KiB） |

**关键规则：**

- **同一时间每个应用只能启用一条**；启用新的，之前启用的自动停用；
- 启用后，正文会**覆盖**目标文件；停用后文件被清空（Pi 的 `AGENTS.md` 会被删除）；
- 文件里已有、但还不在 CC Switch 里的内容，可以用「导入当前的 xxx」存成一条，**不会改动文件**；
- 在别处改了启用中那条对应的文件，下次打开本页时改动会收进启用中的那一条；
- 删除只影响 CC Switch 里的这一条，目标文件不会被改动。

**Pi 的额外内容：**

| 标签 | 说明 |
| :-- | :-- |
| 提示库 | `AGENTS.md` 提示库，同时只启用一条 |
| 系统提示 | 编辑 `APPEND_SYSTEM.md`（追加，推荐）或 `SYSTEM.md`（替换）；文件存在就生效 |
| 模板 | 保存后在 Pi 里输入 `/命令名` 调用 |

**跨应用复制**：预设行 ⋯ 菜单 →「复制到其他应用」，各自复制成独立的一条，不会自动启用。

### 16.3 Skills

**打开**：左侧栏 **Skills**（全局页），两个视图：**已安装** / **发现**。

**支持的应用**：Claude Code、Codex、Gemini CLI、Grok Build、OpenCode、Hermes、Pi、MiniMax Code、Kimi Code、Oh My Pi（OpenClaw 暂不支持）。

> 用户手册 3.3 仍列 8 个应用；README 的功能表已把 **Kimi Code、Oh My Pi** 的 Skills 标为支持。以应用内实际列出的列为准。

**已安装**：矩阵表，每行一个 Skill，每列一个应用，开关控制启用。

**发现**：

- **预配置仓库**：包括 Anthropic 官方 `anthropics/skills` 和 `ComposioHQ/awesome-claude-skills` 等；
- **skills.sh**：公共注册表，至少输入 2 个字，回车搜索。

**安装步骤：**

1. 在「发现」里找到要安装的 Skill；
2. 先在 **「安装到」** 里选好应用（会记住上次选择）；
3. 点这一行的 **「安装」**。

**其他安装方式**（页头「添加 Skill」菜单）：

- **从 ZIP 安装**：选含 `SKILL.md` 的 ZIP；
- **导入本机已有**：把各应用目录里已有、但还不归 CC Switch 管理的 Skill 导入进来（复制一份到 CC Switch 目录，原文件不动）。

**安装位置：**

| 应用 | 目录 |
| :-- | :-- |
| Claude Code | `~/.claude/skills/` |
| Codex | `~/.codex/skills/` |
| Gemini CLI | `~/.gemini/skills/` |
| Grok Build | `~/.grok/skills/` |
| OpenCode | `~/.config/opencode/skills/` |
| Hermes | `~/.hermes/skills/` |
| Pi | `~/.pi/agent/skills/` |
| MiniMax Code | `~/.minimax/skills/` |

**存储与同步**（Skills 页 ⋯ 菜单）：

- **主副本位置**：`~/.cc-switch/skills/`（默认）或通用目录 `~/.agents/skills`；
- **同步方式**：自动 / 只用软链接 / 只用复制。

**卸载**：会先自动创建技能备份（`skill-backups/`，保留最近 20 个），可从备份恢复。

### 16.4 会话管理器

**打开**：左侧栏 **会话**（全局页）。

**支持的应用与会话位置：**

| 应用 | 位置 |
| :-- | :-- |
| Claude Code | `~/.claude/projects/<项目目录>/*.jsonl` |
| Codex | `~/.codex/sessions/` 与 `~/.codex/archived_sessions/` |
| Gemini CLI | `~/.gemini/tmp/<project>/chats/` |
| Grok Build | `~/.grok/sessions/` 与 `~/.grok/archived_sessions/` |
| OpenCode | `~/.local/share/opencode/`（JSON 或 SQLite） |
| OpenClaw | `~/.openclaw/agents/<agent>/sessions/*.jsonl` |
| Hermes | `~/.hermes/state.db` 或 `~/.hermes/sessions/*.jsonl` |
| Pi | `~/.pi/agent/sessions/` |
| MiniMax Code | `~/.minimax/v2/sqlite/runtime-state.sqlite` |

**功能：**

- **列表**：按时间 / 按项目分组；显示标题、活跃时间、最后一条消息摘要；
- **搜索**：标题、项目目录、首末消息、会话 ID；
- **详情**：正文、思考、工具调用、工具结果和图片分开呈现；每一轮折叠成一行摘要（多少步、改了几个文件、几处失败），失败步骤标红；右侧是对话目录；
- **导出**：复制整段为 Markdown，或导出为 Markdown 文件；
- **恢复**：
  - **macOS**：用首选终端启动恢复命令，终端在会话的项目目录中打开；启动失败则复制命令到剪贴板；
  - **其他平台**：复制命令到剪贴板，自己粘贴；
  - OpenClaw 会话由网关管理、Hermes 会话暂不支持命令行恢复；
- **删除**：直接从磁盘删除，**不经过废纸篓，无法撤销**；MiniMax Code 的会话请在 MiniMax Code 中删除。

> **Codex 按供应商分开保存历史**，切换过供应商后，旧会话可能无法继续。

### 16.5 工作区（OpenClaw 专属）

**打开**：左侧栏选 OpenClaw → 页头下的 **「工作区」** 页签。

**工作区文件**：`~/.openclaw/workspace/`

| 文件 | 说明 |
| :-- | :-- |
| `AGENTS.md` | Agent 定义与配置 |
| `SOUL.md` | 系统灵魂/性格设置 |
| `USER.md` | 用户资料信息 |
| `IDENTITY.md` | 身份与角色定义 |
| `TOOLS.md` | 可用工具配置 |
| `MEMORY.md` | 系统记忆 |
| `HEARTBEAT.md` | 心跳配置 |
| `BOOTSTRAP.md` | 引导序列 |
| `BOOT.md` | 启动配置 |

**每日记忆**：`~/.openclaw/workspace/memory/`，按日期命名（`YYYY-MM-DD.md`），支持搜索、编辑、删除。

### 16.6 Hermes 记忆管理

在 Hermes 页面的 **「记忆」** 页里编辑：

- `MEMORY.md`（Agent 记忆）
- `USER.md`（用户画像记忆）

---

## 17. 用量、成本与额度

### 17.1 两个数据来源

| 来源 | 覆盖范围 | 需要本地路由 |
| :-- | :-- | :-- |
| **路由请求日志** | 经本地路由转发的所有请求 | 需要 |
| **CLI 会话日志** | Claude Code、Codex、Gemini CLI、Grok Build、OpenCode、Pi、MiniMax Code 的本地会话记录 | **不需要** |

> **不开本地路由也能统计**：CC Switch 默认定期扫描各工具的本地会话记录。
>
> 会话日志里分不出具体是哪家供应商，这类记录的供应商列显示为「{应用} · 会话日志」。

**扫描节奏：**

- CC Switch 启动时扫描一次；
- 之后后台每 10 分钟扫描一次；
- 打开用量页时立即扫描，并按所选间隔（5 / 10 / 30 / 60 秒，默认 60 秒）继续扫描；
- 关闭「自动扫描会话日志」后，只在点「立即同步」时扫描。

### 17.2 汇总卡片

| 指标 | 说明 |
| :-- | :-- |
| **总成本** | 按定价配置估算的费用 |
| **总请求数** | 统计周期内的请求总数 |
| **真实消耗 Tokens** | 新增输入 + 输出 + 缓存写入 + 缓存命中 |
| **缓存命中率** | 缓存命中 ÷（新增输入 + 缓存写入 + 缓存命中） |

点「更多指标」可以展开分项数字。

> 部分协议（如 OpenAI）不上报缓存写入，缓存写入和真实消耗可能偏低。

### 17.3 时间范围与筛选

| 范围 | 说明 |
| :-- | :-- |
| 当天 | 当天 00:00 至今 |
| 近 24 小时 / 近 7 天 / 近 14 天 / 近 30 天 | 过去对应时长 |
| 全部 | 全部历史 |
| 日历筛选 | 自定义开始和结束 |

可**按应用**、**按供应商**、**按模型**筛选。

> **OpenClaw 和 Hermes 暂不支持用量统计**；Claude Desktop 只统计经本地路由转发的「模型映射」请求，并计入 Claude Code 的筛选。

### 17.4 图表

- **热力图**：最近 53 周每天的用量，颜色越深用量越大；
- **趋势图**：可在 Tokens / 请求 / 成本之间切换；24 小时内按小时，7 天以上按天。

> **缓存 Token 说明**：Anthropic API 支持 Prompt Caching。缓存创建时收取较高费用（通常为输入价格的 1.25 倍），后续命中只收 0.1 倍，可大幅降低重复请求的成本。

### 17.5 请求日志

| 字段 | 说明 |
| :-- | :-- |
| 时间 | 请求时间 |
| 应用 | 所属应用 |
| 供应商 | 供应商名称 |
| 模型 | 请求的模型 |
| 新增输入 | 不含缓存的输入 Token |
| 输出 | 输出 Token |
| 缓存命中 | 缓存命中的 Token |
| 成本 | 估算费用（美元）；没有定价的显示「未定价」 |

> 路由模式下还能看到「请求模型 → 实际模型」，用来核对模型映射是否正确。

### 17.6 额度与余额：两类展示

| 类别 | 覆盖 | 需要手动启用 |
| :-- | :-- | :-- |
| **自动显示** | GitHub Copilot、Codex OAuth 反向代理、xAI OAuth | 否 |
| **手动启用（内置模板）** | 官方订阅（Claude / Codex / Gemini / Grok Build）、Token Plan、第三方余额查询 | **是** |
| **手动启用（自定义脚本）** | 未被内置模板覆盖的中转服务、私有部署 | **是** |

#### 自动显示的行为

- 卡片底部显示额度，一律写「剩余」，如「5 小时剩余 62%」，旁边是重置倒计时；
- 平时灰色，任一档剩余不到 10% 时文字加深加粗；
- 用完显示「已用完」；套餐过期显示「套餐已过期」；查询失败显示「额度没查到」；
- **点击额度文字即可重新查询**；
- 这几类供应商的卡片「更多」菜单里**没有**「配置用量查询」。

#### 手动启用官方订阅额度

v3.16.2 起，Claude / Codex / Gemini / Grok Build 官方供应商的订阅额度改为**默认关闭**的内置模板：

1. 在官方供应商卡片上点右侧 **⋯** → **配置用量查询**；
2. 打开 **「启用用量查询」**；
3. 模板选 **「官方订阅」**；
4. 保存。

> 例外：绑定了授权中心 ChatGPT 账号的 OpenAI Official 供应商**默认开启**。

#### 为什么需要手动启用

**同一个请求地址可能同时提供多种查询模式**——既可能有按套餐的配额查询，也可能有按账户余额的查询。CC Switch 无法自动推断你想查哪一种，所以默认关闭。

#### 内置模板覆盖范围

| 类别 | 覆盖供应商 | 模板类型 |
| :-- | :-- | :-- |
| 官方订阅 | Claude / Codex / Gemini / Grok Build 官方 | 官方订阅配额 |
| Token Plan | Kimi / Zhipu GLM / MiniMax / 火山方舟 / OpenCode Go / Command Code | 套餐配额（带使用进度） |
| 第三方余额 | DeepSeek / StepFun / SiliconFlow / OpenRouter / Novita AI | 官方余额查询 |

#### 自定义脚本

未被内置模板覆盖的供应商，可以用 JavaScript 写查询脚本：

```javascript
({
  request: {
    url: "{{baseUrl}}/user/balance",
    method: "GET",
    headers: {
      "Authorization": "Bearer {{apiKey}}",
      "User-Agent": "cc-switch/1.0"
    }
  },
  extractor: function(response) {
    return {
      isValid: response.is_active || true,
      remaining: response.balance,
      unit: "USD"
    };
  }
})
```

**可用变量**：`{{baseUrl}}`、`{{apiKey}}`、`{{accessToken}}`、`{{userId}}`。

**预设模板**：自定义、通用模板、NewAPI、Sub2API、Token Plan、官方、官方订阅。

**启用后**：

- 自动刷新间隔由「自动查询间隔」控制（设为 `0` 禁用）；
- **仅当供应商处于「使用中」状态时才会触发后台查询**；
- 点「测试脚本」可以先验证。

### 17.7 自定义定价

按模型设置单价，可以从 **models.dev** 导入。

---

## 18. 项目：整套配置一键切换

### 18.1 它是什么

把 Claude Code 或 Codex 当前的**供应商、MCP、Skills 和提示词文件**保存为一个项目（Claude Desktop 只保存供应商）。

### 18.2 开启入口

**设置 → 通用 → 侧栏与页头 → 显示项目切换**（默认关闭）。

打开后，Claude Code、Claude Desktop、Codex 页面顶部会出现项目切换器；托盘菜单里也会出现「项目」。

### 18.3 使用

1. 打开顶部的项目切换器 → **「新建项目」**；
2. 把当前配置保存下来；
3. 之后从切换器里选择即可整套切换；
4. **切到其他项目时，当前状态会自动存回原项目**。

### 18.4 适合的场景

- 不同项目用不同供应商组合；
- 公司网络 vs 家庭网络；
- 不同使用者共用一台电脑。

---

## 19. 数据、备份、云同步与配置文件

### 19.1 数据目录

默认 `~/.cc-switch/`（Windows 为 `C:\Users\<用户名>\.cc-switch`）。

```
~/.cc-switch/
├── cc-switch.db            # SQLite 数据库（SSOT）
├── settings.json           # 设备级设置
├── live-state.json         # 本机状态：各工具是直连还是走路由、上一次写入了什么
├── codex-login-stash.json  # 暂存的 Codex 官方登录
├── copilot_auth.json       # GitHub Copilot 登录凭据
├── codex_oauth_auth.json   # ChatGPT / Codex OAuth 登录凭据
├── xai_oauth_auth.json     # xAI OAuth 登录凭据
├── kimi_accounts.json      # Kimi 账号缓存
├── skills/                 # 技能主副本目录
├── skill-backups/          # 技能备份
├── backups/                # 数据库备份与环境变量备份
│   └── live-first-write/   # 各工具配置文件第一次改写前的原文件
├── logs/                   # cc-switch.log 及轮转文件
└── crash.log               # 崩溃日志
```

> `settings.json`、`live-state.json`、`codex-login-stash.json` 和 `backups/live-first-write/` **只属于这台电脑**：始终在默认目录，不随配置目录移动，也不参与云同步。

### 19.2 数据库内容

`cc-switch.db` 是 SQLite 数据库，主要表：

| 表 | 内容 |
| :-- | :-- |
| `providers` | 供应商配置 |
| `provider_endpoints` | 供应商端点候选列表 |
| `mcp_servers` | MCP 服务器配置 |
| `prompts` | 提示词预设 |
| `skills` / `skill_repos` | 技能安装状态与仓库配置 |
| `profiles` | 项目 |
| `proxy_config` | 本地路由配置（按应用） |
| `proxy_request_logs` | 请求用量明细 |
| `usage_daily_rollups` | 30 天前的用量按日聚合 |
| `provider_health` | 供应商健康状态 |
| `model_pricing` | 模型定价 |
| `settings` | 应用设置 |

### 19.3 备份与恢复

**设置 → 数据 → 备份与恢复：**

| 配置 | 选项 | 默认 |
| :-- | :-- | :-- |
| 自动备份间隔 | 禁用、6h、12h、24h、48h、7d | 24 小时 |
| 备份保留数量 | 3、5、10、15、20、30、50 | 10 个 |

**操作：**

| 操作 | 说明 |
| :-- | :-- |
| 立即备份 | 立即创建一个备份 |
| 恢复 | 从选定的备份恢复数据库；**恢复前会自动创建当前数据库的安全备份** |
| 重命名 | 修改备份的显示名称 |
| 删除 | 永久删除（需确认） |

**「其他备份」** 列出 CC Switch 在各处留下的备份：

- 技能卸载备份；
- 配置文件首次写入备份；
- Codex 会话迁移备份；
- Hermes / OpenClaw 配置备份；
- 环境变量清理备份；
- 旧版遗留文件。

> ⚠️ 有些备份删除后无法恢复对应内容（如删掉「配置文件首次写入备份」后找不到改写前的原文件）。

### 19.4 导出与导入

**导出**：设置 → 数据 → 数据管理 → **「导出 SQL 备份」**

文件名格式：`cc-switch-export-{timestamp}.sql`

包含：所有供应商配置、MCP 服务器配置、Prompts 预设、用量日志、应用设置。

**导入**：

1. 点「选择 SQL 文件」（**只支持 CC Switch 导出的备份**）；
2. 选文件；
3. 点「导入」。

> ⚠️ **导入会覆盖现有配置**，建议先导出当前配置作为备份。导入成功后界面会显示本次导入前自动创建的备份 ID。

> **不包含设备级设置**（`settings.json`，不适合跨设备）。

### 19.5 云同步

**设置 → 数据 → 云同步**，两种方式**同一时间只能启用一种**：

| 方式 | 支持的服务 |
| :-- | :-- |
| **WebDAV** | 坚果云、Nextcloud、群晖 NAS、自定义 |
| **S3 兼容** | AWS S3、MinIO、Cloudflare R2、阿里云 OSS、腾讯云 COS、华为云 OBS、自定义 |

**WebDAV 配置项：**

| 配置 | 说明 |
| :-- | :-- |
| 服务商 | 坚果云 / Nextcloud / 群晖 NAS / 自定义 |
| 服务器地址 | WebDAV URL |
| 账户 | 登录用户名 |
| 密码 | 登录密码（应用专用密码） |
| 远程根目录 | 默认 `cc-switch-sync` |
| 同步配置名 | 默认 `default` |
| 自动同步 | 开启后每次数据库变更都会自动上传 |

**S3 配置项：**

| 配置 | 说明 |
| :-- | :-- |
| 服务商 | AWS S3 / MinIO / R2 / OSS / COS / OBS / 自定义 |
| Region | 如 `us-east-1`；R2 填 `auto` |
| Bucket | 存放同步数据的桶 |
| Access Key ID / Secret Access Key | 访问凭据 |
| Endpoint | AWS 可留空，多数 S3 兼容服务需要填 |
| 远程根目录 | 默认 `cc-switch-sync` |
| 同步配置名 | 默认 `default` |
| 自动同步 | 开启后每次数据库变更都会自动上传 |

**操作：**

| 操作 | 说明 |
| :-- | :-- |
| 测试连接 | 验证地址和凭据 |
| 上传 | 把本地数据库上传到远程 |
| 下载 | 下载远程数据库；下载前显示远程快照信息（协议版本、数据库版本、时间戳、大小）；覆盖本地前会自动创建安全备份 |

> ⚠️ **上传会覆盖远程数据，下载会覆盖本地数据。**

**另一种做法**：把 CC Switch 配置目录放到 Dropbox、OneDrive、iCloud 等网盘文件夹中（设置 → 数据 → 存储位置）。

### 19.6 日志

| 文件 | 内容 |
| :-- | :-- |
| `logs/cc-switch.log` | 应用诊断日志（按 20 MB 轮转，保留最近 4 个归档） |
| `crash.log`、`crash.log.1`、`crash.log.2` | 崩溃日志 |

**设置 → 数据 → 应用诊断日志：**

- **启用应用诊断日志**：控制 `cc-switch.log`；
- **日志级别**：error / warn / info（默认）/ debug / trace。

> 重启应用**不会**清空已有日志。日志可能包含运行环境信息，公开提交前建议先检查内容。

---

## 20. 深度链接与配置分享

### 20.1 它是什么

`ccswitch://` 协议，可以通过链接一键导入配置。适合：

- 团队共享配置；
- 教程中的一键配置；
- 跨设备快速同步。

### 20.2 在线生成器

<https://ccswitch.io/deplink.html>

1. 选择导入类型（供应商 / MCP / 提示词 / Skill 仓库）；
2. 填写配置；
3. 点「生成链接」；
4. 复制分享。

### 20.3 协议格式

```
ccswitch://v1/import?resource={type}&app={app}&name={name}&...
```

**通用参数：**

| 参数 | 必填 | 说明 |
| :-- | :-- | :-- |
| `resource` | 是 | `provider` / `mcp` / `prompt` / `skill` |
| `app` | provider、prompt 需要 | `claude` / `codex` / `gemini` / `agy` / `grokbuild` / `opencode` / `openclaw` / `hermes` / `pi`；prompt 另支持 `mcode` |
| `name` | provider、prompt 必填 | 名称 |

**供应商常用参数：**

| 参数 | 说明 |
| :-- | :-- |
| `endpoint` | API 端点地址（支持逗号分隔多个） |
| `apiKey` | API 密钥 |
| `homepage` | 供应商官网 |
| `model` | 默认模型（Pi 必填） |
| `apiFormat` | `anthropic` / `openai_chat` / `openai_responses` / `gemini_native` |
| `haikuModel` / `sonnetModel` / `opusModel` | 模型档位（仅 Claude） |
| `notes` | 备注 |
| `icon` | 内置图标名 |
| `iconUrl` | 自己的图标地址（v4.0.10+，https、≤256 KB） |
| `config` | Base64 编码的配置内容 |
| `configFormat` | `json` / `toml` |
| `enabled` | 确认框里「导入后立即切换」的初始值 |
| `usageScript` | 用量查询脚本 |
| `usageEnabled` | 是否启用用量查询 |

**示例：**

```
ccswitch://v1/import?resource=provider&app=claude&name=My%20Provider&endpoint=https%3A%2F%2Fapi.example.com&apiKey=sk-xxx
```

> 网页和 GitHub 不可靠地链接自定义协议，站长应链接到 `https://ccswitch.io/import#<参数>`，由它唤起 CC Switch。

### 20.4 安全提示

- 打开深链后，**确认框会显示**将添加的名称、地址、模型和 Key 去向；
- **确认后才保存**；
- 脚本正文会完整展示在导入确认框中；
- 不要导入来源不明的链接。

---

## 21. 环境变量冲突

### 21.1 为什么会冲突

**环境变量的优先级通常高于配置文件**，可能导致：

- CC Switch 设置的供应商配置被覆盖；
- API 请求发送到错误的端点；
- 使用错误的 API 密钥。

### 21.2 检测范围

| 应用 | 检测范围 |
| :-- | :-- |
| Claude Code | 以 `ANTHROPIC` 开头的变量 |
| Codex | 以 `OPENAI` 开头的变量 |
| Gemini CLI | 以 `GEMINI`、`GOOGLE_GEMINI` 开头的变量 |
| Grok Build | `XAI_API_KEY`、`GROK_DEFAULT_MODEL` |

### 21.3 处理

检测到冲突时，界面顶部显示黄色警告横幅：

```
⚠️ 检测到系统环境变量冲突
发现 X 个环境变量可能会覆盖您的配置
[查看详情] [×]
```

**处理步骤：**

1. 点「查看详情」查看变量名、值和来源；
2. 勾选要删除的变量（或「全选」）；
3. 点「删除选中」；
4. 在确认对话框里点「确认删除」；
5. **重启应用或终端**才能生效。

**自动备份**：删除前会备份到 `~/.cc-switch/backups/env-backup-<时间戳>.json`，可在「其他备份」里查看。

**手动处理：**

- **Windows**：系统属性 → 高级 → 环境变量 → 删除或修改；
- **macOS / Linux**：编辑 `~/.zshrc` / `~/.bashrc`，注释掉相关 `export`，然后 `source ~/.zshrc`。

### 21.4 最佳实践

1. **用 CC Switch 管理配置**：避免在系统环境变量里设 API 密钥；
2. **定期检查**冲突警告；
3. **备份重要变量**。

---

## 22. 工具安装与升级

### 22.1 应用页

左侧栏底部的 **「应用」** 页：

- 查看每个工具的**当前版本**和**最新版本**；
- 安装未装的；
- 升级已装的；
- **全部升级**；
- 选择哪些应用显示在侧栏；
- **「手动安装命令」**：复制各工具的安装命令。

> **Claude Desktop 是桌面应用，不能在这里安装或升级**，页面只显示它是否已接入。
>
> 启动时是否检查新版本由「设置 → 通用 → 各应用的新版本」控制；**不会自动升级**。

### 22.2 诊断重复安装

应用页可以诊断重复安装（例如同一工具通过 npm 和官方安装器各装了一份）。

### 22.3 WSL 里的工具

CC Switch **不会自动识别 WSL**。在 Windows 上：

1. **设置 → 应用配置** → 把对应工具的配置目录改成 WSL 里的路径，例如：

```
\\wsl.localhost\Ubuntu\home\<用户名>\.claude
```

2. 保存后 CC Switch 就会读写 WSL 里的配置；
3. 设置之后，「应用」页也会在对应的 WSL 发行版里检测和升级该工具。

**支持的 8 个应用**：Claude Code、Codex、Gemini CLI、Grok Build、OpenCode、OpenClaw、Hermes、Pi。

> ⚠️ **路由和聚合写入配置的地址是 `127.0.0.1`。** WSL2 默认的 NAT 网络模式下，WSL 里的 `127.0.0.1` 连不到 Windows 上的本地路由，需要改用 WSL 的 **mirrored 网络模式**。

---

## 23. 设置项全表

### 23.1 通用

| 设置 | 说明 |
| :-- | :-- |
| **界面语言** | 简体中文、繁體中文、English、日本語，切换后立即生效 |
| **明暗** | 跟随系统 / 浅色 / 深色 |
| **在侧栏显示哪些应用** | 至少保留一个 |
| **显示项目切换** | 默认关闭 |
| **启动时检查各应用的新版本** | 只提示，不自动升级 |
| **开机自启** | Windows 注册表 / macOS LaunchAgent / Linux XDG autostart |
| **静默启动** | 开机自启后多出的开关：启动时不显示主窗口 |
| **关闭时最小化到托盘** | 默认开启 |
| **启用应用级窗口按钮（仅 Linux）** | 用应用自绘的窗口按钮 |
| **首选终端** | macOS：Terminal、iTerm2、Alacritty、Kitty、Ghostty、Otty、WezTerm、Kaku、Warp；Windows：命令提示符、PowerShell、Windows Terminal；Linux：GNOME Terminal、Konsole、Xfce4 Terminal、Alacritty、Kitty、Ghostty |
| **轻量模式** | 见 [6.6](#66-轻量模式) |

### 23.2 应用配置

**Claude Code：**

| 开关 | 作用 |
| :-- | :-- |
| **应用到 Claude Code 插件** | VS Code 插件的供应商随 CC Switch 切换（写入 `~/.claude/config.json` 的 `primaryApiKey`） |
| **跳过 Claude Code 初次安装确认** | 在 `~/.claude.json` 写入 `hasCompletedOnboarding: true`；关闭时删除该字段 |

**Codex 应用增强：**

| 开关 | 作用 |
| :-- | :-- |
| **非接管切换时保留官方登录** | 关闭（默认）：切到第三方时删除 `~/.codex/auth.json`（删除前暂存到 `codex-login-stash.json`）；开启：保留 |
| **统一 Codex 会话历史** | 开启后官方与第三方会话出现在同一个历史列表；跨供应商继续旧会话时对方后端可能无法解密推理内容 |
| **Codex 官方额度保护**（v4.0.25） | 官方 5 小时/周额度达到阈值（默认 95% / 98%）时暂停经路由的官方请求，避免 Codex 掉进 Reserve 模式 |

**配置目录覆盖（8 个应用）：**

| 应用 | 默认 |
| :-- | :-- |
| Claude Code | `~/.claude/` |
| Codex | `~/.codex/` |
| Gemini CLI | `~/.gemini/` |
| Grok Build | `~/.grok/` |
| OpenCode | `~/.config/opencode/` |
| OpenClaw | `~/.openclaw/` |
| Hermes | `~/.hermes/` |
| Pi | `~/.pi/agent/` |

> Claude Desktop 和 MiniMax Code **不支持**覆盖目录。
>
> 修改目录后需要**重启应用**，且对应的 CLI 工具也需要配置相同的目录。

### 23.3 本地路由

见 [10.4](#104-路由配置) 与 [11](#11-故障转移与熔断)。

### 23.4 网络

**全局出站代理**：配置 CC Switch 访问外部 API 时使用的 HTTP/HTTPS 代理。

### 23.5 数据

见 [19](#19-数据备份云同步与配置文件)。

### 23.6 关于

- 版本信息、检查更新；
- GitHub、官方网站、发行说明；
- 「近期更新」：重新查看最近几个版本的更新摘要。

---

## 24. 常见问题 FAQ

### Q1：切换供应商后需要重启终端吗？

| 工具 | 是否需要重启 |
| :-- | :-- |
| **Claude Code** | 不需要（支持热切换） |
| **Codex、Gemini CLI、Grok Build** | 需要重启终端或 CLI 工具 |
| **Claude Desktop** | 需要完全退出并重新打开 |
| **OpenCode、OpenClaw、Hermes、Pi、MiniMax Code、Kimi Code、Oh My Pi** | 共存式，点「添加」后写入配置，在工具里选模型即可 |

### Q2：切换供应商会改掉我的插件、Hook 等设置吗？

**不会。** 只替换关键字段：请求地址、Key、模型名和接口协议（Codex 还包括推理档位，Gemini CLI 还包括认证方式），以及少数跟着供应商走的兼容选项。

插件、Hook、权限、MCP、你自己加的环境变量、注释和排版都原样保留。

> 以前的「通用配置片段」已经不需要了，相关按钮已移除。

### Q3：在工具里换了模型，切走再切回来怎么又变回去了？

模型属于关键字段，归供应商所有。想长期使用某个模型，**请在 CC Switch 里编辑这个供应商**。

### Q4：为什么总有一个正在使用的供应商无法删除？

设计原则是「最小侵入性」：即使卸载 CC Switch，也不影响应用正常使用。所以对切换式应用，系统总会保留一个正在使用的配置。

想删它，**先切换到别家**。

### Q5：如何切换回官方登录？

1. 在供应商列表里找到自带的官方供应商（Claude Official、Claude Desktop Official、OpenAI Official、Google Official、Grok Official）；
2. 点「启用」（**路由模式下不能切到官方供应商，先点「回到直连」**；Codex 的 OpenAI Official 卡片除外）；
3. 重启对应的 CLI 工具；
4. 按工具自身的登录流程操作（如 Claude Code 的 `/login`、Codex 的 `codex login`）。

### Q6：开始路由后，配置文件里的地址为什么变成了 127.0.0.1？

路由和聚合模式下，工具的请求先发到 CC Switch 的本地路由（默认 `http://127.0.0.1:15721`），再转发给你选中的供应商。所以配置文件里只有本地地址和占位密钥 `PROXY_MANAGED`。

真实地址、Key 和模型都保存在 CC Switch 里。

### Q7：能在 Claude Code 里使用 OpenAI 兼容接口、Gemini 或本地模型吗？

**可以，但需要使用路由。**

1. 编辑供应商，在「高级选项 → 上游格式」里选择和供应商一致的接口格式；
2. 在 Claude Code 页面的「路由」标签里点「开始路由」。

格式选错或没有开始路由，通常会报 **404 或 405**。

### Q8：「检测连通」通过了，为什么请求还是失败？

「检测连通」**只检查地址能不能连上**，不会发送真实模型请求，所以验证不了 API Key 和模型名是否正确。

请求失败时，请检查 Key、模型名和上游格式；经过路由的请求还可以在「用量统计」的请求日志里查看具体报错。

### Q9：我的数据存储在哪里？

默认在 `~/.cc-switch/`（Windows 为 `C:\Users\<用户名>\.cc-switch`），详见 [19.1](#191-数据目录)。

### Q10：在 Windows 上怎么管理 WSL 里的工具？

见 [22.3](#223-wsl-里的工具)。

### Q11：有命令行版本或无界面版本吗？

官方只提供**需要图形界面的桌面版**。

服务器、SSH 远程场景推荐社区维护的 **[CC Switch CLI](https://github.com/SaladDay/cc-switch-cli)**：

- 提供交互式终端界面（TUI）和命令行两种用法；
- 支持 Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw、Hermes、Pi；
- 通过 `brew install cc-switch-cli` 或安装脚本安装；
- 默认与桌面版共用数据目录 `~/.cc-switch`，也兼容桌面版的 WebDAV 同步。

> 两个项目分别发版，CLI 版支持的数据库版本有时会落后于桌面版。遇到「数据库版本过新」的提示时，升级 CLI 版或等它跟进。

### Q12：Linux（Wayland + NVIDIA）网页内容点不动、缩放后黑屏？

见 [4.4](#44-linux)。

### Q13：macOS 安装提示「无法验证开发者」？

CC Switch macOS 版本**已通过 Apple 代码签名和公证**，正常情况不会出现。如遇问题，请从 Releases 页面下载最新版本。

### Q14：Windows 安装后无法启动？

1. 安装 [Microsoft Edge WebView2](https://developer.microsoft.com/en-us/microsoft-edge/webview2/)；
2. 把 CC Switch 加入杀毒软件白名单。

### Q15：托盘图标不显示？

| 系统 | 处理 |
| :-- | :-- |
| macOS | 检查系统设置中的菜单栏图标设置 |
| Windows | 检查任务栏设置，确保图标未被隐藏 |
| Linux | 安装系统托盘支持（如 `libappindicator`） |

### Q16：Linux 标题栏按钮点不到？

1. **设置 → 通用 → 窗口与终端 → 启用应用级窗口按钮**；
2. 或用托盘菜单的「退出」结束应用；
3. AppImage 用户可设 `CC_SWITCH_GDK_BACKEND=x11`。

### Q17：界面显示异常？

1. 切换主题（浅色/深色）；
2. 重启应用；
3. 删除 `~/.cc-switch/settings.json` 重置设置（**各应用当前使用的供应商指针也记在这个文件里，重置后需要重新确认**）。

### Q18：更新失败？

1. 检查网络连接；
2. 手动下载最新版本安装；
3. Homebrew 安装的：`brew upgrade --cask cc-switch`。

### Q19：导入配置失败？

1. 确认文件是 **CC Switch 导出的 SQL 备份文件**；
2. 检查文件内容是否完整；
3. 尝试用文本编辑器打开检查格式。

### Q20：配置丢失？

1. 检查 `~/.cc-switch/` 目录是否存在（如果改过配置目录，以改后的为准）；
2. 从备份恢复：设置 → 数据 → 备份与恢复；
3. 或从之前导出的 SQL 文件导入。

### Q21：为什么有的供应商自动显示配额，有的需要手动启用？

只有 **OAuth 账号类**供应商（GitHub Copilot、Codex OAuth、xAI OAuth）会在启用后自动显示配额。

**其他所有供应商**（包括官方订阅、Token Plan 和第三方余额）都需要手动到卡片「配置用量查询」里打开「启用用量查询」并选择内置模板。

原因：同一个请求地址可能同时有「套餐」和「余额」两种查询模式，需要你自行选择。

### Q22：官方订阅供应商没有显示配额？

1. 确认供应商处于「使用中」状态（**未使用时不触发查询**）；
2. 对于 Copilot / Codex OAuth，检查 OAuth Token 是否仍在有效期内；
3. 检查网络连通性；
4. 点击卡片上的刷新图标手动重新查询。

### Q23：Codex 聚合模式下，`/model` 里看不到聚合的模型？

见 [12.7](#127-什么时候需要重启)。

### Q24：CC Switch 支持哪些 AI 工具？

**十二个**：Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build、OpenCode、OpenClaw、Hermes、Pi、MiniMax Code、Kimi Code、Oh My Pi。

---

## 25. 故障排查手册

### 25.1 先做这四件事

1. **看卡片状态**：是不是「使用中」？
2. **看模式**：直连 / 路由 / 聚合，哪个在生效？
3. **看日志**：`~/.cc-switch/logs/cc-switch.log`；
4. **看用量**：用量统计 → 请求日志，有没有这次请求、报了什么错。

### 25.2 切换后不生效

| 检查 | 处理 |
| :-- | :-- |
| 是否重启了工具 | Codex / Gemini / Grok Build / Claude Desktop 都需要重启 |
| 是否改对了应用 | 检查左侧栏选的是不是那个工具 |
| 配置目录是否被覆盖 | 设置 → 应用配置 → 配置目录 |
| 是否有环境变量冲突 | 看顶部黄色警告 |

### 25.3 请求返回 401 / 403

| 原因 | 处理 |
| :-- | :-- |
| Key 错误或过期 | 重新填 Key |
| 认证字段选错 | Claude 走 OpenAI 兼容网关时，保持 `ANTHROPIC_AUTH_TOKEN`；改成 `ANTHROPIC_API_KEY` 会以 `x-api-key` 发送，多数网关不接受 |
| OAuth Token 失效 | 到授权中心重新登录 |
| 余额不足 | 检查供应商后台 |

### 25.4 请求返回 404 / 405

| 原因 | 处理 |
| :-- | :-- |
| **上游格式选错** | 在「高级选项 → 上游格式」里改成和供应商一致 |
| **没开路由** | 非原生格式必须开本地路由 |
| **base URL 拼接错误** | 试试「完整 URL」模式 |
| **模型名不存在** | 用「获取模型列表」或查供应商文档 |

### 25.5 请求超时

1. 检查网络；
2. 尝试直接访问供应商 API（关闭路由）；
3. 检查供应商配置；
4. 调整超时参数（设置 → 本地路由 → 自动故障转移）。

### 25.6 本地路由启动失败

**端口被占用：**

```bash
# macOS / Linux
lsof -i :15721

# Windows
netstat -ano | findstr :15721
```

**处理：**

1. 关闭占用端口的程序；
2. 或在「设置 → 本地路由 → 服务」里先停止路由服务（还有应用在用路由时，先让它们回到直连），改用其他端口（1024–65535），点「保存并重启服务」。

### 25.7 回到直连后配置未恢复

**可能原因：** CC Switch 异常退出、配置文件被其他程序修改。

**处理：**

1. 重新打开 CC Switch，它启动时会按本机状态恢复各工具的配置；
2. 或在应用页点「回到直连」；在「设置 → 本地路由」里点「全部回到直连」；
3. 仍不对，检查当前供应商的端点地址；也可以从 `~/.cc-switch/backups/live-first-write/` 找回原配置文件。

### 25.8 故障转移没有触发

检查清单：

- [ ] 应用是否在**路由**模式（聚合模式不做故障转移）
- [ ] 本地路由服务是否运行
- [ ] 「路由」页签里的自动故障转移开关是否打开
- [ ] 队列中是否有备用供应商

### 25.9 频繁触发故障转移

1. 检查主供应商状态；
2. **调高失败阈值**（如从 3 改为 5）；
3. 考虑更换主供应商。

### 25.10 所有供应商都熔断了

1. 等待熔断时长到期（默认 60 秒）；
2. 或在「设置 → 本地路由 → 服务」里停止后重新启动路由服务，重置状态；
3. 或调整熔断器参数。

### 25.11 用量统计数据为空

- [ ] 「自动扫描会话日志」是否开启，对应 CLI 是否有会话历史
- [ ] 如果依赖路由请求日志：本地路由是否运行、应用是否在路由或聚合模式、「记录请求用量」是否开启
- [ ] 该应用是否支持用量统计（OpenClaw、Hermes 不支持）

### 25.12 Codex 用量和直连时对不上

v3.13.0 起，Codex 用量从估算切换为**基于 JSONL 会话日志的精确解析**，并对模型名称做归一化以保证定价一致。新数据会与官方账单对齐；若仍看到旧的估算数据，可以删除历史条目或等待新会话数据覆盖。

### 25.13 深链接点了没反应

- 确认已安装 CC Switch 并注册了协议；
- 网页和 GitHub 不可靠地链接自定义协议，请用 `https://ccswitch.io/import#<参数>`；
- 轻量模式下会按需重建主窗口，第一次会略慢。

### 25.14 想彻底重来

1. **设置 → 数据 → 数据管理 → 导出 SQL 备份**（先备份）；
2. 逐个应用切回官方供应商，或直接卸载；
3. 删除 `~/.cc-switch/`；
4. 重新安装、重新添加供应商。

> 删除前先确认各工具能正常工作——卸载 CC Switch 后，配置文件会停留在最后一次写入的状态。

---

## 26. 卸载

### 26.1 Windows

- 通过「设置 → 应用」卸载；
- 或运行安装目录下的卸载程序。

### 26.2 macOS

```bash
# Homebrew 安装的
brew uninstall --cask cc-switch

# 加 --zap 会同时删除配置数据
brew uninstall --zap --cask cc-switch
```

手动安装的：把 `CC Switch.app` 移到废纸篓。

可选：删除配置目录 `~/.cc-switch/`。

### 26.3 Linux

```bash
# Debian/Ubuntu
sudo apt remove cc-switch

# Fedora 等 RPM
sudo dnf remove cc-switch

# Arch
paru -R cc-switch-bin
```

### 26.4 卸载前要做的事

1. **切回官方供应商**，或确认当前供应商配置正确——卸载后配置文件停留在最后一次写入的状态；
2. **导出 SQL 备份**（如果需要保留配置）；
3. **记下 OAuth 账号**——卸载不会让它们在任何地方退出登录，但本机凭据会丢。

> CC Switch 的设计原则是「最小侵入性」：**即使卸载，也不影响应用的正常使用**。

---

## 27. 术语表

| 术语 | 含义 |
| :-- | :-- |
| **供应商（Provider）** | 一份连接信息：名称、地址、Key、模型、上游格式 |
| **切换式应用** | 同一时间只启用一个供应商（Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build） |
| **共存式应用** | 多个供应商同时写入配置（OpenCode、OpenClaw、Hermes、Pi、MiniMax Code、Kimi Code、Oh My Pi） |
| **直连** | 客户端直接连供应商 |
| **本地路由** | CC Switch 在本机起的 HTTP 服务（默认 `127.0.0.1:15721`） |
| **聚合模式** | 多家供应商的模型放进同一个模型列表 |
| **上游格式** | 供应商说的是哪种 API（Anthropic / OpenAI Chat / OpenAI Responses / Gemini Native） |
| **模型映射** | 把 Claude Code 的模型角色映射到上游真实模型 |
| **故障转移** | 主供应商失败时自动换下一个 |
| **熔断器** | 连续失败到阈值后暂时跳过某供应商 |
| **整流器** | 自动修复部分上游不兼容的请求 |
| **项目（Profile）** | 供应商 + MCP + Skills + 提示词的命名快照 |
| **授权中心** | 统一管理 OAuth 账号 |
| **统一供应商** | 一份配置同步到 Claude Code / Codex / Gemini CLI |
| **深度链接** | `ccswitch://` 协议，一键导入配置 |
| **`PROXY_MANAGED`** | 路由/聚合模式下写进配置文件的占位密钥 |
| **`ccs-` 前缀** | 聚合模式下非默认供应商的模型前缀 |
| **`live-state.json`** | 记录各工具当前模式的文件，**不要手动改** |

---

## 附录 A：各工具配置文件速查

### A.1 CC Switch 自身

| 路径 | 内容 |
| :-- | :-- |
| `~/.cc-switch/cc-switch.db` | SQLite 主数据库 |
| `~/.cc-switch/settings.json` | 设备级设置 |
| `~/.cc-switch/live-state.json` | 本机模式状态 |
| `~/.cc-switch/codex-login-stash.json` | 暂存的 Codex 官方登录 |
| `~/.cc-switch/copilot_auth.json` | Copilot 凭据 |
| `~/.cc-switch/codex_oauth_auth.json` | ChatGPT OAuth 凭据 |
| `~/.cc-switch/xai_oauth_auth.json` | xAI OAuth 凭据 |
| `~/.cc-switch/skills/` | 技能主副本 |
| `~/.cc-switch/skill-backups/` | 技能备份 |
| `~/.cc-switch/backups/` | 数据库备份 |
| `~/.cc-switch/backups/live-first-write/` | 配置文件首次改写前的原文件 |
| `~/.cc-switch/logs/cc-switch.log` | 应用日志 |
| `~/.cc-switch/crash.log` | 崩溃日志 |

### A.2 各工具

| 工具 | 配置目录 | 主文件 | 提示词 | MCP | Skills | 会话 |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| Claude Code | `~/.claude/` | `settings.json` | `CLAUDE.md` | `~/.claude.json` | `skills/` | `projects/<项目>/*.jsonl` |
| Claude Desktop | `Claude/` + `Claude-3p/` | `claude_desktop_config.json` | – | – | – | – |
| Codex | `~/.codex/` | `config.toml` + `auth.json` | `AGENTS.md` | `config.toml` 的 `[mcp_servers]` | `skills/` | `sessions/`、`archived_sessions/` |
| Gemini CLI | `~/.gemini/` | `.env` + `settings.json` | `GEMINI.md` | `settings.json` 的 `mcpServers` | `skills/` | `tmp/<project>/chats/` |
| Grok Build | `~/.grok/` | `config.toml` | `AGENTS.md` | `config.toml` 的 `[mcp_servers]` | `skills/` | `sessions/`、`archived_sessions/` |
| OpenCode | `~/.config/opencode/` | `opencode.json` | `AGENTS.md` | `opencode.json` 的 `mcp` | `skills/` | `~/.local/share/opencode/` |
| OpenClaw | `~/.openclaw/` | `openclaw.json`（JSON5） | 工作区编辑器 | – | – | `agents/<agent>/sessions/*.jsonl` |
| Hermes | `~/.hermes/` | `config.yaml` | `SOUL.md` | `config.yaml` 的 `mcp_servers` | `skills/` | `state.db` 或 `sessions/*.jsonl` |
| Pi | `~/.pi/agent/` | `models.json` + `settings.json` | `AGENTS.md`、`SYSTEM.md` | `mcp.json` | `skills/` | `sessions/` |
| MiniMax Code | `~/.minimax/` | `config.yaml` | `AGENTS.md`（≤32 KiB） | `mcp.json` | `skills/` | `v2/sqlite/runtime-state.sqlite` |
| Kimi Code | `~/.kimi-code/` | `config.toml` | `AGENTS.md` | `mcp.json` | `skills/` | `sessions/<workDirKey>/<sessionId>/` |
| Oh My Pi | `~/.omp/agent/`（命名 profile 为 `~/.omp/profiles/<name>/agent/`） | `models.yml`（写供应商）+ `config.yml`（默认模型） | `AGENTS.md` | `mcp.json` | 由 Pi 体系管理 | `sessions/` |

> 环境变量覆盖：`CLAUDE_CONFIG_DIR`、`CODEX_HOME`、`HERMES_HOME`、`PI_CODING_AGENT_DIR`、`MINIMAX_DATA_DIR`（或 `MAVIS_DATA_DIR`）、`KIMI_CODE_HOME`。

### A.3 路由/聚合模式下写入的地址

| 工具 | 地址 |
| :-- | :-- |
| Claude Code | `http://127.0.0.1:15721` |
| Codex | `http://127.0.0.1:15721/v1` |
| Gemini CLI | `http://127.0.0.1:15721` |
| Grok Build | `http://127.0.0.1:15721/grokbuild/v1` |
| Claude Desktop | `http://127.0.0.1:15721/claude-desktop` |

密钥统一写 `PROXY_MANAGED`。

---

## 附录 B：默认参数速查

| 项 | 默认值 |
| :-- | :-- |
| 本地路由监听地址 | `127.0.0.1` |
| 本地路由端口 | `15721` |
| 记录请求用量 | 开启 |
| 数据目录 | `~/.cc-switch/` |
| 自动备份间隔 | 24 小时 |
| 备份保留数量 | 10 个 |
| 技能备份保留 | 20 个 |
| 日志轮转 | 20 MB，保留 4 个归档 |
| 熔断失败阈值 | 通用 4 / Claude 8 |
| 熔断恢复等待 | 通用 60 秒 / Claude 90 秒 |
| 流式首字节超时 | 通用 60 秒 / Claude 90 秒 |
| 流式静默超时 | 通用 120 秒 / Claude 180 秒 |
| 非流式超时 | 600 秒 |
| 最大重试次数 | 通用 3 / Claude 6 / Gemini 5 |
| 连通检测超时 | 8 秒 |
| 连通检测较慢阈值 | 6000 毫秒 |
| 用量扫描间隔（页面打开时） | 60 秒 |
| 用量后台扫描间隔 | 10 分钟 |
| 远程同步根目录 | `cc-switch-sync` |

---

## 附录 C：上线前检查清单

- [ ] 已备份各工具的现有配置（CC Switch 也会自动备份到 `live-first-write/`）
- [ ] CC Switch 已安装并能启动，托盘图标正常
- [ ] 侧栏只显示你实际使用的应用（减少干扰）
- [ ] 至少添加了一个供应商，并点「检测连通」确认地址可达
- [ ] 切换后重启了对应工具（Claude Code 除外）
- [ ] 发一句测试消息能正常回复
- [ ] 「用量统计」里能看到这次请求
- [ ] 如使用路由：本地路由已启动，端口未被占用
- [ ] 如使用路由：理解「CC Switch 必须保持运行，退出会自动回到直连」
- [ ] 如使用聚合：确认 Codex 已彻底重启，`/model` 里能看到聚合模型
- [ ] 如使用故障转移：队列已按优先级排好，熔断参数符合预期
- [ ] 如使用订阅当供应商：已阅读并接受风险提示
- [ ] 环境变量冲突横幅已处理或已确认忽略
- [ ] 已开启自动备份，或手动导出过 SQL 备份
- [ ] 知道日志位置（`~/.cc-switch/logs/cc-switch.log`）
- [ ] 知道如何切回官方供应商（官方卡 + 重启 + 工具自己的登录流程）
- [ ] 知道「正在使用的供应商不能删除」，需要先切走

---

> **最后一句：** CC Switch 的核心只有三步——**加供应商 → 切换 → 重启工具**。路由、聚合、故障转移、MCP / Skills 管理都是在你需要时再逐层加上的可选能力。遇到问题先看**卡片状态**、**当前模式**、**用量统计的请求日志**和 `cc-switch.log`，大多数问题都能定位。
