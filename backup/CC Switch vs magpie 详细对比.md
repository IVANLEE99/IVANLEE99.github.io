> **一句话：** 两者都能「让 Claude Code 用别家的模型」，但**根本不是同一类工具**——CC Switch 是**配置管理器**（帮你改配置文件，路由是可选增强），magpie 是**本地网关**（所有请求必经它转发，配置只是副产品）。

---

## 关于本文

- **对比对象**：
  - **CC Switch** — `farion1231/cc-switch`，版本基准 **v4.0.25**（2026-11-02）
  - **magpie** — `yetone/magpie`，版本基准 **v0.1.2829**（2026-11-03）
- **资料来源**：两者的官方中文 README、官方中文用户手册 / 文档站、英文完整参考文档、官方使用攻略、CHANGELOG。
- **写作原则**：
  - 只写官方文档能支撑的事实；
  - 官方文档没写的，明确标注「文档未说明」，不臆测；
  - 两者都迭代很快，**具体菜单文案和参数以应用内实际显示为准**。
- **重要提醒**：本文涉及「用订阅当供应商」「OAuth 反向代理」等能力，两者都有，也都**可能违反厂商服务条款**。对比不代表推荐，风险请自行评估。

---

## 目录

- [0. 30 秒结论](#0-30-秒结论)
- [1. 本质差异：一个是配置管理器，一个是网关](#1-本质差异一个是配置管理器一个是网关)
- [2. 总览对照表](#2-总览对照表)
- [3. 逐维度详解](#3-逐维度详解)
  - [3.1 技术形态与部署](#31-技术形态与部署)
  - [3.2 支持的 Agent 数量](#32-支持的-agent-数量)
  - [3.3 供应商与预设](#33-供应商与预设)
  - [3.4 配置写入方式](#34-配置写入方式)
  - [3.5 请求路径：直连 vs 必经网关](#35-请求路径直连-vs-必经网关)
  - [3.6 路由能力](#36-路由能力)
  - [3.7 故障转移](#37-故障转移)
  - [3.8 会话粘性](#38-会话粘性)
  - [3.9 多模型并存（聚合 vs 路由组）](#39-多模型并存聚合-vs-路由组)
  - [3.10 协议转换与模型映射](#310-协议转换与模型映射)
  - [3.11 订阅与账号管理](#311-订阅与账号管理)
  - [3.12 MCP 管理](#312-mcp-管理)
  - [3.13 Skills 与提示词](#313-skills-与提示词)
  - [3.14 会话管理](#314-会话管理)
  - [3.15 用量与成本](#315-用量与成本)
  - [3.16 额度与余额](#316-额度与余额)
  - [3.17 隐私与脱敏](#317-隐私与脱敏)
  - [3.18 遥测](#318-遥测)
  - [3.19 多机共享与远程](#319-多机共享与远程)
  - [3.20 命令行与自动化](#320-命令行与自动化)
  - [3.21 插件与扩展](#321-插件与扩展)
  - [3.22 数据、备份与同步](#322-数据备份与同步)
  - [3.23 安全模型](#323-安全模型)
  - [3.24 许可与社区](#324-许可与社区)
- [4. 场景选型指南](#4-场景选型指南)
- [5. 组合使用：两者能一起用吗](#5-组合使用两者能一起用吗)
- [6. 从一方迁移到另一方](#6-从一方迁移到另一方)
- [7. 各自的限制与坑](#7-各自的限制与坑)
- [8. 版本与时效说明](#8-版本与时效说明)
- [附录 A：端口与路径对照](#附录-a端口与路径对照)
- [附录 B：术语对照](#附录-b术语对照)
- [附录 C：命令与入口对照](#附录-c命令与入口对照)

---

## 0. 30 秒结论

| 你的情况 | 选它 |
| :-- | :-- |
| 只想**换个 API Key / 换个中转站**，不想开后台进程 | **CC Switch** |
| 想在**一个客户端里同时看到多家模型**，`/model` 里直接切 | **CC Switch 聚合模式** |
| 想要**复杂的路由策略**（按意图、按文件夹、按时段、按成本、按周额度配速） | **magpie** |
| 想让**提示词里的密钥不发给厂商** | **magpie**（本地脱敏，独有） |
| 想在 **NAS / 服务器 / Docker** 上跑一个共享网关 | **magpie** |
| 想**在终端里**完成所有操作（TUI / CLI） | **magpie** |
| 想管理 **MCP / Skills / 提示词 / 会话阅读** | **CC Switch**（矩阵式管理更成熟） |
| 想**同一台电脑上给多个 Claude 订阅换号** | **CC Switch**（切卡即切号） |
| 想**跨协议**在 Claude Code 里用 GPT、在 Codex 里用 Claude | 两者都行，CC Switch 教程更细，magpie 配置更简单 |
| 你只有 **1 个 Agent + 1 个供应商** | 两者都不必要 |

**一句话取舍**：

> - **CC Switch = 「我要改配置，顺便有个路由」** → 轻、可退出、桌面优先。
> - **magpie = 「我要一个网关，配置顺手就改了」** → 重、常驻、策略强、可上服务器。

---

## 1. 本质差异：一个是配置管理器，一个是网关

### 1.1 CC Switch 的心智模型

```
你 → CC Switch 界面 → 改写各工具的配置文件 → 工具自己直连厂商
```

- **主产品是「配置文件的可视化管理」**；
- 切换供应商 = 改写 `settings.json` / `config.toml` / `.env` 里的关键字段；
- **路由是可选功能**：只有你想跨协议或要故障转移时，才进入路由模式；
- **直连模式下，CC Switch 可以退出**，工具照常工作（配置已经写进去了）。

### 1.2 magpie 的心智模型

```
你 → magpie 界面 → 改写各工具的配置指向 127.0.0.1:3425 → 网关 → 厂商
```

- **主产品是「本地网关」**；
- 给 Agent 选模型 = 把 Agent 的 base URL 指向网关 + 写入 `provider/model`；
- **所有请求必经网关**（这是设计前提，不是可选项）；
- **magpie 必须一直运行**（或装成后台服务 `magpie daemon install`），否则所有 Agent 不可用。

### 1.3 这个差异带来的连锁效应

| 连锁效应 | CC Switch | magpie |
| :-- | :-- | :-- |
| 应用能否退出 | **能**（直连模式下配置已落盘） | **不能**（退出则网关消失） |
| 是否需要后台服务 | 不需要 | 建议 `magpie daemon install` |
| 请求是否可被记录 | 仅路由模式下 | **始终** |
| 延迟 | 直连无额外延迟 | 每个请求多一跳本机转发 |
| 能否跨协议 | 仅路由模式下 | **始终支持**（网关原生转换） |
| 配置复杂度 | 低（改几个键） | 中（要理解 provider/model 与路由组） |

> **这条差异决定了一切**：如果你只想要「一键换 Key」，CC Switch 更轻；如果你想要「统一入口 + 强策略」，magpie 更合适。

---

## 2. 总览对照表

### 2.1 基本盘

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **定位** | 配置管理器 + 可选本地路由 | 本地网关 + 配置管理 |
| **开源协议** | MIT | MIT |
| **版本线** | v4.0.25（2026-11） | v0.1.2829（2026-11） |
| **技术栈** | Tauri 2 + Rust + React 18 + TS + SQLite | Go + 平台 WebView + SQLite/JSON |
| **支持的 Agent 数** | **12 个**（精挑细选） | **70+ 个**（全覆盖） |
| **供应商预设** | **90+** | **27+ 列出 + 任意 OpenAI/Anthropic 兼容** |
| **网关/路由端口** | `127.0.0.1:15721`（仅路由模式） | `127.0.0.1:3425`（始终） |
| **配置目录** | `~/.cc-switch/` | `~/.config/magpie/` |
| **官方 CLI** | ❌ 无（社区版 `cc-switch-cli`） | ✅ 完整 CLI |
| **TUI** | ❌ | ✅ `magpie tui` |
| **Web 界面** | ❌（只有桌面窗口） | ✅ `magpie web` |
| **Docker** | ❌ | ✅ `ghcr.io/yetone/magpie` |
| **后台服务** | ❌ | ✅ `magpie daemon install` |

### 2.2 能力矩阵

图例：✅ 完整支持　🟡 部分支持 / 有条件　❌ 不支持

| 能力 | CC Switch | magpie |
| :-- | :--: | :--: |
| 一键切换供应商 | ✅ | ✅ |
| 系统托盘快速切换 | ✅（5 个切换式应用） | ✅（菜单栏/托盘） |
| 配置方案 / Profile | ✅（项目） | ✅（方案） |
| 本地路由 / 网关 | ✅（可选） | ✅（必须） |
| 跨协议转换 | ✅（路由模式） | ✅（始终） |
| 自动故障转移 | ✅（队列 + 熔断器） | ✅（路由组 + 账号切换） |
| 按权重/成本/速度路由 | ❌ | ✅ |
| 按意图/文件夹/时段路由 | ❌ | ✅ |
| 会话粘性控制 | 🟡（路由模式下自动） | ✅（5 档可调） |
| 多模型同列（聚合） | ✅（Claude Code / Codex） | 🟡（路由组近似） |
| 嵌套路由组 | ❌ | ✅（最多 8 层） |
| 订阅登录当供应商 | ✅（Copilot / ChatGPT / xAI / Kimi / Claude） | ✅（Claude / ChatGPT / Copilot / Gemini / Grok） |
| Claude 多账号切换 | ✅（切卡即切号） | ✅（多账号 + 自动切换） |
| MCP 管理 | ✅（矩阵表 + 按工具同步） | ✅（资料库 + MCP router） |
| Skills 管理 | ✅（发现 / 安装 / 更新 / 备份） | ✅（资料库） |
| 提示词管理 | ✅（按工具，Markdown 编辑器） | ✅（资料库） |
| 会话管理 | ✅（阅读页、搜索、导出、恢复） | 🟡（会话列表 + 花费 + 重开） |
| 用量统计 | ✅（会话日志 + 路由日志） | ✅（网关记录） |
| 额度/余额显示 | ✅（内置模板 + 自定义 JS 脚本） | ✅（订阅额度 + 余额） |
| 上下文占用分析 | ❌ | ✅ |
| **本地脱敏（密钥不出本机）** | ❌ | ✅ |
| 联网搜索（任意模型） | ❌ | ✅ |
| 生图 / 生视频 | ❌ | ✅ |
| 插件系统 | ❌（仅用量脚本 JS） | ✅（OpenCode 插件 + 网关中间件） |
| 深度链接导入 | ✅ `ccswitch://` | ✅ `magpie://import` |
| 环境变量冲突检测 | ✅ | ❌ |
| 多机共享网关 | ❌ | ✅（远程 magpie） |
| 云同步配置 | ✅（WebDAV / S3） | ✅（WebDAV / S3 / GitHub） |
| 备份恢复 | ✅（SQL） | ✅（加密备份文件） |
| OTLP 导出 | ❌ | ✅ |
| Docker / NAS | ❌ | ✅ |
| WSL 支持 | 🟡（手动填路径） | 🟡（原生安装 + 会话统计） |
| 遥测 | 文档未说明 | ✅ 每日匿名计数（可关） |

---

## 3. 逐维度详解

### 3.1 技术形态与部署

#### CC Switch

| 项 | 说明 |
| :-- | :-- |
| **形态** | 纯桌面应用（Tauri 2） |
| **平台** | Windows 10+、macOS 12+、Linux（glibc 2.35+ / WebKitGTK 4.1） |
| **安装** | MSI / Portable / DMG / Homebrew / deb / rpm / AppImage / AUR |
| **是否需要运行时** | 不需要 Node.js |
| **能否无界面运行** | ❌ 不能（官方只提供图形界面版） |
| **服务器场景** | 推荐社区版 [CC Switch CLI](https://github.com/SaladDay/cc-switch-cli) |
| **资源占用** | 桌面应用量级；轻量模式可销毁窗口降到接近零 |

#### magpie

| 项 | 说明 |
| :-- | :-- |
| **形态** | 桌面应用 + 终端版 + Web 版 + 网关 + 后台服务 |
| **平台** | macOS、Windows、Linux、Docker、Termux |
| **安装** | `install.sh` / Homebrew 类脚本 / `go install` / `make build` / Docker |
| **是否需要运行时** | 单二进制，无外部运行时（插件需要时会自动下载 Bun） |
| **能否无界面运行** | ✅ `magpie serve`、`magpie daemon install` |
| **服务器场景** | ✅ Docker + Web 界面（网关模式） |
| **资源占用** | Go 二进制；网关常驻内存 |

> **结论**：**服务器 / NAS / SSH / 纯终端场景，magpie 是唯一选择**；CC Switch 必须依赖图形桌面。

---

### 3.2 支持的 Agent 数量

#### CC Switch：12 个，深度适配

Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build、OpenCode、OpenClaw、Hermes、Pi、MiniMax Code、Kimi Code、Oh My Pi。

**特点**：

- 每个工具都有**专属的配置写入逻辑**（知道每个工具该改哪些键）；
- 有**功能矩阵**（哪些工具支持 MCP / Skills / 提示词 / 会话 / 用量）；
- 切换式（5 个）与共存式（7 个）区分明确；
- **只显示已安装或配置过的工具**。

#### magpie：70+ 个，广度覆盖

Claude Code、Claude Desktop、Codex、Gemini CLI、Antigravity CLI、OpenCode、OpenChamber、MiMo Code、DevEco Code、Pi、oh-my-pi、StepCode、MothX、PenguinHarness、Aside、OmO、Prime Agent、Goose、Cursor CLI、Zed、VS Code Chat、VSCodium Chat、JetBrains Air、Copilot CLI、Crush、DeepSeekRuntime、Command Code、fx、Devin、Hermes Agent、Kimi Code、jcode、Qwen Code、Muse Code、Empryo、MiniMax Code、Droid、Cline、Qoder、Grok Build、ZCode、WorkBuddy、CodeBuddy、Raycast、Kiro Crew、T3 Code、OpenHanako、AtomCode、Snow CLI、Alma、OpenClaw、AstrBot、QwenPaw……

**特点**：

- 官方参考文档有**逐 Agent 的文件 / 字段对照表**；
- 支持**任意能填 base URL 的工具**（直接指向网关）；
- 只显示本机已装的 Agent；
- 有些 Agent 需要重启才生效（文档逐条说明）。

> **结论**：**你用的工具冷门 → magpie**；**你只用主流 5 个 → CC Switch 适配更深**（尤其是 Claude Desktop、Codex 的特殊选项）。

---

### 3.3 供应商与预设

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **预设数量** | 90+ | 27+ 列出 + 任意兼容地址 |
| **分类方式** | 账号登录 / 模型厂商 / 第三方平台 / 云服务商 / 插件配置 | 订阅 / 厂商 / 本机 / 自定义 |
| **搜索** | ✅ 按中文名和域名 | ✅ 按名称 |
| **同厂商多套餐合并** | ✅ 一行内切换套餐/地区/版本 | 🟡 分开列出 |
| **自动拉取模型列表** | ✅ `/v1/models` | ✅ 向厂商询问 |
| **模型别名** | 🟡（模型映射表，Codex） | ✅ `magpie model id` 任意重命名 |
| **统一供应商**（一份配置给多工具） | ✅ 跨 Claude Code / Codex / Gemini | ✅（本来就是一套 provider 给所有 Agent） |
| **预设快照问题** | ⚠️ 旧卡不自动升级 | 🟡 类似（需刷新 provider） |
| **端点测速** | ✅ 多端点 + 延迟颜色分级 | ❌（有 provider test 单点延迟） |
| **自定义图标** | ✅ 内置 + 自上传 | 🟡（供应商图标） |

**CC Switch 的独特之处**：

- **统一供应商**：一份配置同步到 Claude Code / Codex / Gemini CLI，改一处全同步；
- **端点测速面板**：一个供应商配多个端点，测完选最快的；
- **预设快照提醒**：文档明确警告「旧卡不会自动升级」。

**magpie 的独特之处**：

- **模型别名**：把 `grok-4.6@500k` 这种怪名字映射成 `grok-4.6`，且能参与自动组；
- **自动组**：两家供应商有同名模型时，自动合并成 `auto/<模型>` 路由组；
- **provider 级 header / User-Agent 覆盖**：某些编程套餐只认特定 UA。

---

### 3.4 配置写入方式

#### 都只改「关键字段」

两者都会保留插件、Hook、权限、MCP、注释和排版。

| 工具 | CC Switch 改什么 | magpie 改什么 |
| :-- | :-- | :-- |
| Claude Code | `env` 的 `ANTHROPIC_*`、`AWS_*`、`model`、`apiKeyHelper` | `env` 的 `ANTHROPIC_BASE_URL`、`ANTHROPIC_AUTH_TOKEN`、模型变量 |
| Codex | `model_provider`、`model`、推理档位、`[model_providers.custom]` | `[model_providers.magpie]` 表 + 模型 |
| Gemini CLI | `.env` 关键行 + `security.auth.selectedType`、`model.name` | `GOOGLE_GEMINI_BASE_URL`、认证方式、模型 |
| Grok Build | `[models] default` + `[model."<名称>"]` 表 | 模型、推理档位 |

#### 备份策略

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **首次改写前备份** | ✅ `~/.cc-switch/backups/live-first-write/` | ✅ `stash.json` 记录被替换的值 |
| **能否一键还原** | ✅ 切回官方 / `default` 供应商 | ✅ `magpie <agent> default` |
| **原子写入** | ✅ | ✅ |
| **格式错误时** | ✅ 拒绝写入并提示行号 | ✅ 原子写入 |

---

### 3.5 请求路径：直连 vs 必经网关

这是**最影响日常体验**的差异。

#### CC Switch

```
直连模式：  Claude Code ──────────────────────► api.anthropic.com
路由模式：  Claude Code ──► 127.0.0.1:15721 ──► 供应商
聚合模式：  Claude Code ──► 127.0.0.1:15721 ──► 按模型分发到不同供应商
```

- **默认是直连**；
- 只有点了「开始路由」或「切换到聚合模式」才走本地；
- 路由模式下配置文件写 `PROXY_MANAGED` 占位符；
- **退出 CC Switch 会自动写回直连配置**。

#### magpie

```
始终：      Claude Code ──► 127.0.0.1:3425 ──► 供应商
```

- **没有「直连模式」**（对 Agent 而言）；
- Agent 配置永远指向网关；
- 网关退出 = Agent 全部不可用；
- 用 `magpie daemon install` 解决常驻问题。

#### 实际影响

| 场景 | CC Switch | magpie |
| :-- | :-- | :-- |
| 忘记开应用 | 直连模式照常工作 | **全部 Agent 报错** |
| 出差/关机 | 配置停留在最后状态 | 需要服务在跑 |
| 延迟 | 直连零额外延迟 | 每请求多一跳（本机，通常 <10ms） |
| 请求记录 | 仅路由模式 | 全部 |
| 换供应商的即时性 | 直连需重启工具 | 网关切换立即生效（模型变了仍需重启） |

---

### 3.6 路由能力

#### CC Switch：简单直接

- **四种连接模式**：直连 / 路由 / 聚合（+ Claude Desktop 的模型映射）；
- **上游格式转换**：Anthropic ↔ OpenAI Chat ↔ OpenAI Responses ↔ Gemini Native；
- **整流器**：自动修复部分上游不兼容请求（如 Thinking 签名、不支持图片时降级）；
- **路由设置**：按应用独立配置超时、重试、熔断。

**没有的能力**：

- 按意图选模型；
- 按文件夹选模型；
- 按时段选模型；
- 按成本选模型；
- 按权重分配；
- 按周额度配速。

#### magpie：策略极其丰富

| 路由模式 | 作用 |
| :-- | :-- |
| `smart` | 还有余量的订阅里，额度最先重置的排第一 |
| `order` | 先用第一个，答不了再换下一个 |
| `drain` | 一个用到额度用完再换，前面恢复了也不切回 |
| `rotate` | 每一轮换下一个成员 |
| `usage` | 剩余额度最多的排第一 |
| `pace` | 每周剩余额度 ÷ 距重置小时数，最高的排第一 |
| `weight` | 按权重分配（1–1000），均匀穿插 |
| `fast` | 最近回答最快的先用 |
| `cheap` | 此刻每百万 token 价格最低的先用 |

**规则（rules）还能按条件分流**：

| 条件 | 例子 |
| :-- | :-- |
| `tokens ≥ N` | 长对话优先发给强模型 |
| 带图片 | 图片任务交给多模态模型 |
| 要求推理 | 推理请求给 Opus |
| 来自某些 Agent | 只在 Claude Code 里生效 |
| **意图** | 用小模型判断「这是简单问题还是写测试」 |
| **压缩上下文** | `/compact` 的总结交给便宜模型 |
| **文件夹** | `~/work/acme` 里的对话用公司模型 |
| **时段/星期** | 工作时间不用贵的账号 |
| **推理强度** | 指定该模型用什么 effort |

**还有**：

- **嵌套路由组**（最多 8 层，禁止循环）；
- **意图路由**（小模型判断，超时可配）；
- **自动组**（同名模型自动合并）。

> **结论**：**路由策略 magpie 完胜**。CC Switch 的路由是「够用」，magpie 的路由是「专业」。

---

### 3.7 故障转移

#### CC Switch：队列 + 熔断器

| 项 | 说明 |
| :-- | :-- |
| **触发条件** | 应用处于路由模式 + 打开故障转移开关 + 队列有成员 |
| **队列** | 按优先级，P1 → P2 → P3 |
| **熔断器** | 失败阈值、恢复阈值、恢复等待、错误率阈值、最小请求数 |
| **默认阈值** | 通用 4 次 / Claude 8 次 |
| **超时** | 流式首字节、流式静默、非流式 |
| **重试** | 通用 3 次 / Claude 6 次 / Gemini 5 次 |
| **健康状态** | 🟢 正常 / 🟡 降级 / 🔴 熔断 |
| **限制** | **聚合模式不做故障转移** |

#### magpie：路由组 + 账号级切换

| 项 | 说明 |
| :-- | :-- |
| **触发条件** | 用 `group/<id>` 选模型 |
| **层级** | 模型级 + 账号级（同一 provider 多账号自动切换） |
| **策略** | 9 种（见上一节） |
| **会话粘性** | 5 档（auto / session / turn / off / strict） |
| **熔断** | 文档未提供用户可调的熔断参数（内部有休息机制） |
| **限制** | 只有一个账号且没建路由组时，没有可换的，错误原样返回 |

> **结论**：**CC Switch 的熔断参数更可调**（适合精细运维）；**magpie 的切换策略更多样**（适合多账号/多订阅）。

---

### 3.8 会话粘性

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **可配置性** | 🟡 路由模式下自动保持（`auto` 语义） | ✅ 5 档可选 |
| **选项** | 无显式选项 | `auto` / `session` / `turn` / `off` / `strict` |
| **strict** | ❌ | ✅ 绝不换账号，错误直接给 Agent |
| **原理** | 靠会话 ID | 靠 `affinity.json` 记录 |

> 如果你有多账号且关心「一个对话别在账号间跳来跳去导致缓存失效」，**magpie 的 `strict` 很有用**。

---

### 3.9 多模型并存（聚合 vs 路由组）

这是**两者最容易被混淆**的地方。

#### CC Switch 聚合模式

**效果**：Claude Code / Codex 的 `/model` 列表里**同时出现多家供应商的模型**。

```
/model
├── gpt-5.6-sol          （默认供应商）
├── ccs-deepseek-v4-pro  （DeepSeek）
├── ccs-kimi-k3          （Kimi）
└── ccs-grok-4.6         （Grok）
```

- 选哪个模型，请求就发给哪一家；
- **不需要回 CC Switch 切换**；
- 非默认供应商的模型带 `ccs-` 前缀；
- **不做故障转移**；
- 只支持 **Claude Code 和 Codex**；
- Claude Code 需要 2.1.243+。

#### magpie 路由组

**效果**：Agent 把**一个组**当作一个模型来选。

```
Agent 选择：group/daily-coding
       ↓
magpie 网关按策略在组内成员之间分发
```

- Agent 看到的是 `group/<id>`，**看不到组内各成员**；
- 按 `smart` / `order` / `weight` 等策略分发；
- 支持故障转移；
- 支持嵌套；
- 所有 Agent 都能用（不限 Claude Code / Codex）。

#### 关键区别

| 维度 | CC Switch 聚合 | magpie 路由组 |
| :-- | :-- | :-- |
| **谁来选** | **你**在 `/model` 里选 | **网关**按策略自动选 |
| **Agent 看到** | 多个具体模型 | 一个组名 |
| **故障转移** | ❌ | ✅ |
| **适用 Agent** | 仅 Claude Code / Codex | 全部 |
| **适合场景** | 想手动对比不同模型 | 想自动分配、保可用性 |

> **一句话**：**聚合是「把选择权交给你」，路由组是「把选择权交给策略」。**

---

### 3.10 协议转换与模型映射

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **支持协议** | Anthropic Messages / OpenAI Chat / OpenAI Responses / Gemini Native | 同左 |
| **转换触发** | 仅路由 / 聚合模式 | 始终（网关自动） |
| **Claude → OpenAI** | ✅（上游格式选 Chat 或 Responses） | ✅ |
| **Codex → Anthropic** | ✅（上游格式选 Anthropic Messages） | ✅ |
| **模型映射** | ✅ Codex 的映射表（显示名 / 实际模型 / 上下文窗口 / 思考等级） | ✅ `magpie model id` 别名 |
| **Claude 三档映射** | ✅ Sonnet / Opus / Haiku 各自指定 | ✅ opus / sonnet / haiku 分档 |
| **上下文窗口声明** | ✅ `1M` 复选框、`model_context_window` | ✅ 各 Agent 配置里写 |
| **思考等级转换** | ✅ `reasoning.effort` 映射 | ✅ 自动折算 |
| **完整 URL 模式** | ✅（非标准路径） | 🟡（自定义 base URL） |
| **整流器** | ✅（自动修复不兼容请求） | 🟡（有兼容处理，文档未单列） |

> **CC Switch 的模型映射 UI 更直观**（表格填写）；**magpie 的别名机制更适合批量**（一条命令改一批）。

---

### 3.11 订阅与账号管理

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **Claude 订阅** | ✅ 多账号（v4.0.21+，切卡即切号） | ✅ 多账号 + 自动切换 |
| **ChatGPT / Codex** | ✅ Codex OAuth（Device Code） | ✅ 订阅登录 |
| **GitHub Copilot** | ✅ | ✅ |
| **Gemini** | 🟡（Google Official） | ✅ Gemini Code Assist |
| **xAI / Grok** | ✅（OAuth） | ✅ SuperGrok |
| **Kimi Code** | ✅ 多账号 | 🟡（通过插件） |
| **Antigravity** | ❌ | ✅ |
| **多账号自动切换** | 🟡（手动切换 / 托盘） | ✅ 自动（`smart` / `pace` 等） |
| **额度显示** | ✅ 卡片 + 托盘 | ✅ `magpie quota` + 卡片 |
| **登录位置** | `~/.cc-switch/*_auth.json` | `~/.config/magpie/logins.json` |
| **风险提示** | ✅ 文档有专门章节 | ✅ 文档有专门说明 |

#### 各自的特色

**CC Switch**：

- **「切卡即切号」**：给每张官方卡绑一个 Claude 账号，点卡片就换号，**正在运行的会话不用重启**；
- **「一个窗口一个号」**：从卡片「打开终端」，该终端固定用这个账号（本质是 `CLAUDE_CONFIG_DIR`）；
- **合规对照表**：逐条对照 Anthropic 官方要求，说明为什么这套做法合规；
- **限制**：不支持 Claude Desktop、不支持 WSL。

**magpie**：

- **多账号自动切换**：不需要手动切，网关按策略分配；
- **每日预热**：在选定时间启动 Claude / ChatGPT 账号的 5 小时窗口；
- **`magpie accounts switch claude next`**：命令行切到下一个还有额度的账号；
- **限制**：Claude 订阅请求通过本机 Claude Code 运行，所以要保留 Claude Code。

---

### 3.12 MCP 管理

#### CC Switch：矩阵式

```
        Claude  Codex  Gemini  Grok  OpenCode  Hermes  Pi  MiniMax
fetch      ✅     ✅     ✅     ✅      ✅       ✅    ✅    ✅
time       ✅     ❌     ✅     ✅      ✅       ✅    ✅    ❌
memory     ✅     ✅     ❌     ✅      ✅       ✅    ✅    ✅
```

- **每行一个 MCP，每列一个应用**，开关控制同步；
- 支持**从应用导入**已有配置；
- 支持**模板**（fetch / time / memory / sequential-thinking / context7）；
- 支持**粘贴配置**自动识别格式；
- 支持 **Deep Link 导入**；
- **连通性检查**（v4.0.25 新增，30 秒超时，最多 3 并发）。

#### magpie：资料库 + MCP router

- **资料库**：MCP 服务器只写一次，magpie 按每个 Agent 的格式写进它的文件；
- **MCP router**：把资料库里的多个 MCP 服务器**合成一个** `magpie` 入口（`/mcp/magpie/hub`），Agent 只配一个；
- 每个工具名格式为 `<server>__<tool>`；
- 命令在首次调用时启动，10 分钟无调用后结束。

#### 对比

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **管理界面** | 矩阵表（直观） | 资料库列表 |
| **按工具开关** | ✅ 一格一个开关 | ✅ 勾选目标 Agent |
| **从现有配置导入** | ✅ | 🟡 |
| **聚合多个 MCP** | ❌ | ✅ MCP router |
| **连通性检查** | ✅ | ❌ |
| **模板** | ✅ | ❌ |

---

### 3.13 Skills 与提示词

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **Skills 管理** | ✅ 发现（skills.sh + GitHub 仓库）/ 安装 / 更新 / 卸载 / 备份 | ✅ 资料库 |
| **Skills 存储** | `~/.cc-switch/skills/` 或 `~/.agents/skills`，软链接或复制同步 | 资料库目录 |
| **提示词管理** | ✅ 按工具，Markdown 编辑器，同时只启用一条 | ✅ 资料库（指令文件） |
| **提示词写入** | `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` | 各 Agent 的指令文件 |
| **提示词导入** | ✅ 「导入当前的 xxx」 | 🟡 |
| **Pi 特殊支持** | ✅ `APPEND_SYSTEM.md` / `SYSTEM.md` / 模板 | ❌ |

> **CC Switch 的 Skills 生态更成熟**（直接搜 skills.sh、检查更新、一键全部更新）；**magpie 的资料库更强调「一处定义、多处写入」**。

---

### 3.14 会话管理

#### CC Switch：阅读器级

- **列表**：按时间 / 按项目分组，显示标题、活跃时间、最后一条消息摘要；
- **搜索**：标题、项目目录、首末消息、会话 ID；
- **详情**：正文、思考、工具调用、工具结果、图片**分开呈现**；
- **摘要**：每一轮折叠成一行（多少步、改了几个文件、几处失败），**失败步骤标红**；
- **目录**：右侧对话目录，可跳转；
- **查找**：会话内查找；
- **导出**：复制整段为 Markdown，或导出文件；
- **恢复**：macOS 一键在终端恢复；其他平台复制命令；
- **删除**：批量删除（**不可撤销**）。

#### magpie：轻量级

- **会话列表**：每个 Agent 对话的花费和标题；
- **一键重开**：在终端里重新打开；
- **上下文分析**：每个请求的上下文窗口用了多满、多少来自缓存、被什么占着（**这个 CC Switch 没有**）。

> **CC Switch 是「会话阅读器」，magpie 是「会话成本视图」**。

---

### 3.15 用量与成本

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **数据来源** | 会话日志（不开路由）+ 路由日志（开路由） | 网关记录（全部） |
| **能否统计未走网关的请求** | ✅（扫描会话文件） | ❌（只统计经过网关的） |
| **Token 分类** | 新增输入 / 输出 / 缓存写入 / 缓存命中 | token / 缓存读写 / 推理 / 调用次数 |
| **成本计算** | ✅（按定价配置） | ✅（按官方价格或自定义） |
| **自定义定价** | ✅ 按模型设置，可从 models.dev 导入 | ✅ 任何模型可自定义价格 |
| **图表** | 热力图（53 周）+ 趋势图（Tokens/请求/成本） | 用量页 + 上下文分析 |
| **请求日志** | ✅ 逐条（时间/应用/供应商/模型/token/成本） | ✅ 逐条（路由页 + 用量页） |
| **按应用筛选** | ✅ | ✅ |
| **按供应商/模型筛选** | ✅ | ✅ |
| **会话成本** | ✅ | ✅ |
| **CSV 导出** | 🟡（SQL 备份含用量） | ✅ `magpie usage --csv` |
| **OTLP 导出** | ❌ | ✅（Langfuse 等） |
| **上下文占用分析** | ❌ | ✅ |
| **缓存命中率** | ✅ | ✅ |
| **输出速度** | ✅ | ✅ |

> **magpie 的「上下文分析」是独有功能**：能告诉你提示词里系统提示词、工具定义、指令文件、文件读取各占多少，还有百分制得分和「MCP 偏重」这类标签。

---

### 3.16 额度与余额

#### CC Switch

| 类别 | 是否需要手动启用 |
| :-- | :-- |
| **自动显示**：GitHub Copilot、Codex OAuth、xAI OAuth | ❌ 不需要 |
| **手动启用（内置模板）**：官方订阅、Token Plan、第三方余额 | ✅ 需要 |
| **手动启用（自定义脚本）**：未覆盖的中转站 | ✅ 需要 |

- 卡片底部显示「5 小时剩余 62%」+ 重置倒计时；
- 剩余不到 10% 时文字加深加粗；
- **支持自定义 JS 脚本查询**（`{{baseUrl}}`、`{{apiKey}}` 等变量）；
- 预设模板：通用模板、NewAPI、Sub2API。

#### magpie

- `magpie quota` 查看每个套餐剩余；
- 额度卡片：5 小时 / 周 / 月窗口 + 重置时间；
- **每日预热**：在选定时间启动 Claude / ChatGPT 的 5 小时窗口；
- **额度刷新**：默认自动，可设「仅手动」；
- **共享 magpie 的额度**：本机显示共享端的最近读取结果。

#### 对比

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **自动显示范围** | 3 类 OAuth 账号 | 订阅 + Key 余额 |
| **自定义查询** | ✅ JS 脚本（强） | 🟡 文档未说明可自定义 |
| **每日预热** | ❌ | ✅ |
| **额度等待** | ❌ | ✅ `magpie quota wait` |
| **按密钥限额** | ❌ | ✅ 网关密钥可设限额 |

---

### 3.17 隐私与脱敏

**这是 magpie 的独有护城河。**

#### magpie 本地脱敏

- **默认关闭**，在 设置 → 隐私 打开；
- 请求离开电脑前，把 API Key、私钥、Token、密码、邮箱、身份证、银行卡等换成 `{{API_KEY_k3v9x2mq}}` 占位符；
- 回复里的占位符**再换回原值**（流式文本和工具调用参数都算）；
- **同一个值每次占位符相同**，所以对话上下文、提示词缓存、thinking 签名都不受影响；
- **原值只在内存**，磁盘记录只留类型、长度和开头几个字符；
- **失败即不发送**（返回 503），绝不退回成不脱敏；
- **可选用本地模型**查找规则找不到的人名、地址（AstrLink Guard 约 39 MB / Privacy Filter 约 1.6 GB）；
- **可按 provider 跳过**（本地 Ollama、LM Studio 等）。

**不能遮的**：思考内容（厂商签名）、ID 和工具定义、图片和文件数据。

#### CC Switch

**没有脱敏功能。** 官方文档未提供任何「请求内容脱敏」能力。

#### 对比

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **密钥脱敏** | ❌ | ✅ |
| **个人信息脱敏** | ❌ | ✅ |
| **自定义词** | ❌ | ✅ |
| **本地模型查找** | ❌ | ✅ |
| **按 provider 跳过** | ❌ | ✅ |
| **失败保护** | — | ✅ 503 不发送 |

> 如果你经常让 Agent 读 `.env`、配置文件、带 token 的日志，**magpie 的脱敏是刚需**。

---

### 3.18 遥测

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **官方文档说明** | **未提及** | ✅ 详细列出 |
| **发送内容** | 文档未说明 | 随机 ID、版本、系统、用了哪些 agent/provider/model |
| **是否发送密钥/提示词** | 文档未说明 | ❌ 明确不发送 |
| **关闭方式** | — | 设置 → 隐私 / `DO_NOT_TRACK=1` / `MAGPIE_NO_STATS=1` |
| **源码构建** | — | ✅ 从不发送 |

> ⚠️ **注意**：CC Switch 官方文档未说明遥测情况，**不能据此断定「没有遥测」**。关注隐私的话，建议自行检查源码或抓包。magpie 的遥测行为是**公开、可关、有文档**的。

---

### 3.19 多机共享与远程

#### CC Switch：配置同步，不共享网关

- **云同步**：WebDAV / S3 兼容存储，同步数据库；
- 每台机器**独立运行**，各自连各自的供应商；
- 没有「把另一台电脑当网关」的能力。

#### magpie：真正的远程网关

- **局域网共享**：一台机器开启共享，生成网关密钥；
- **远程 magpie**：其他电脑把它添加为 provider（`office/…`）；
- 共享端保存供应商、订阅、密钥、路由组；
- 本机仍运行 magpie，只借用模型；
- **Docker + NAS**：在服务器上跑一个 magpie，给全屋设备供模型。

#### 对比

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **配置同步** | ✅ | ✅ |
| **网关共享** | ❌ | ✅ |
| **多机密钥管理** | ❌ | ✅（每客户端一个密钥，可限额） |
| **服务器部署** | ❌ | ✅ Docker |
| **离线可用** | ✅（各机独立） | ❌（共享端离线则不可用） |

---

### 3.20 命令行与自动化

#### CC Switch

- **官方无 CLI**；
- 社区版 [CC Switch CLI](https://github.com/SaladDay/cc-switch-cli) 提供 TUI + 命令行；
- 支持 Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw、Hermes、Pi；
- 与桌面版共用 `~/.cc-switch`。

#### magpie

完整 CLI：

```sh
magpie ls                              # 列出每个 Agent 及设置
magpie providers                       # 你的 provider
magpie models                          # 所有可选模型
magpie claude deepseek/deepseek-v4-pro # 设置模型
magpie save work && magpie use work    # 方案
magpie quota                           # 额度
magpie usage 7d                        # 用量
magpie group add ...                   # 路由组
magpie daemon install                  # 后台服务
magpie backup / restore                # 备份恢复
magpie tool generate_image "..."       # 生图
magpie tui / web / serve / tray        # 多种运行方式
```

#### 对比

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **官方 CLI** | ❌ | ✅ |
| **TUI** | ❌ | ✅ |
| **Web 界面** | ❌ | ✅ |
| **脚本化** | 🟡（社区 CLI） | ✅ |
| **CI / 自动化** | 🟡 | ✅ |

---

### 3.21 插件与扩展

#### CC Switch

- **没有插件系统**；
- 只有**用量查询脚本**（JavaScript，自定义 API 查询逻辑）。

#### magpie

- **插件 = provider 插件**：兼容 OpenCode 的 provider 插件，也支持 pi 扩展包；
- 插件负责登录、列模型、发请求；
- 运行在 Bun 上（首次使用时自动下载）；
- **网关中间件**：`onRequest` / `onEvent` / `onResponse` 钩子，可改写或拒绝请求；
- **现成中间件**：`param-override`、`model-map`、`system-prompt`、`word-guard`、`think-tags`；
- 社区插件库：`magpie-community/plugins`。

**中间件示例**：

```js
export function onRequest(body, ctx) {
  if (body.model === "fast") return { ...body, model: "deepseek/deepseek-chat" };
}
```

#### 对比

| 维度 | CC Switch | magpie |
| :-- | :-- | :-- |
| **插件系统** | ❌ | ✅ |
| **请求中间件** | ❌ | ✅ |
| **自定义登录** | ❌ | ✅ |
| **生态** | 无 | ✅ 社区插件库 |

---

### 3.22 数据、备份与同步

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **主存储** | SQLite `cc-switch.db` | `settings.json` + `providers.json` 等 |
| **配置目录** | `~/.cc-switch/` | `~/.config/magpie/` |
| **自动备份** | ✅ 默认 24 小时，保留 10 个 | 🟡 需手动 `magpie backup` |
| **备份加密** | ❌（SQL 明文） | ✅ AES-256-GCM + 口令 |
| **导出格式** | `.sql` | `.magpie-backup` |
| **云同步** | ✅ WebDAV / S3 | ✅ WebDAV / S3 / **GitHub** |
| **同步冲突** | 上传覆盖远程 / 下载覆盖本地 | ✅ 条件写，自动合并 |
| **日志** | `logs/cc-switch.log`（20 MB 轮转，保留 4 个） | `app.log`（5 MiB 轮转，保留 2 个） |

> **magpie 的备份是加密的**，CC Switch 的 SQL 备份是明文的（含 API Key）。**备份文件请妥善保管**。

---

### 3.23 安全模型

#### CC Switch

| 项 | 说明 |
| :-- | :-- |
| **监听范围** | 默认 `127.0.0.1`，可改 `0.0.0.0` 允许局域网 |
| **外部访问认证** | 无内置认证（靠监听地址限制） |
| **密钥存储** | 明文在 SQLite 中 |
| **日志脱敏** | 文档未说明 |

#### magpie

| 项 | 说明 |
| :-- | :-- |
| **监听范围** | 默认 `127.0.0.1` |
| **外部访问认证** | ✅ 网关密钥（`sk-magpie-key-…`），可按密钥设限额和模型白名单 |
| **IP 白名单** | ✅ `MAGPIE_ALLOW_IPS` |
| **TLS** | ✅ `MAGPIE_TLS_CERT` / `MAGPIE_TLS_KEY` |
| **密钥存储** | `providers.json` 权限 `0600` |
| **日志脱敏** | ✅ 看起来像密钥的内容会被遮掉 |
| **本地脱敏** | ✅ 见 3.17 |

> **如果要暴露到局域网，magpie 的安全设计完整得多**（密钥 + 限额 + IP 白名单 + TLS）。**CC Switch 的本地路由没有认证机制**，只适合本机使用。

---

### 3.24 许可与社区

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **许可** | MIT | MIT |
| **作者** | farion1231（Jason Young） | yetone |
| **仓库** | `farion1231/cc-switch` | `yetone/magpie` |
| **是否接受 PR** | ✅ 接受（需先开 Issue 讨论） | ❌ **不接受 PR** |
| **反馈渠道** | GitHub Issues | Discord |
| **文档语言** | 中文 / 英文 / 日文 | 中文 / 英文 / 日文 |
| **发布节奏** | 很频繁（v4.0.x 每天多个补丁） | 很频繁（0.1.x） |
| **官网** | ccswitch.io | usemagpie.ai |

> **magpie 不接受 PR** 是一个重要的社区差异——想要功能只能去 Discord 提。

---

## 4. 场景选型指南

### 4.1 决策树

```
你主要想解决什么？
│
├─ 只想换 API Key / 换中转站，越简单越好
│   └─► CC Switch
│
├─ 想在 Claude Code 的 /model 里同时看到 GPT、Kimi、DeepSeek
│   └─► CC Switch 聚合模式
│
├─ 想让 Claude Code 用 GPT（或 Codex 用 Claude）
│   ├─ 想要图文教程一步步来 ──► CC Switch
│   └─ 想要命令行 + 策略路由 ──► magpie
│
├─ 有 3 个以上 Claude/ChatGPT 账号，想自动分配
│   └─► magpie（路由组 + 账号切换）
│
├─ 想按「这个问题简单还是复杂」自动选模型
│   └─► magpie（意图路由）
│
├─ 想让提示词里的 .env 内容不发给厂商
│   └─► magpie（本地脱敏）
│
├─ 想在 NAS 上跑一个网关，全家设备共用
│   └─► magpie（Docker + 远程 magpie）
│
├─ 主要用终端，讨厌 GUI
│   └─► magpie（TUI / CLI）
│
├─ 想管理 MCP / Skills / 提示词 / 看会话记录
│   └─► CC Switch
│
├─ 想给多个 Claude 订阅「切卡即切号」
│   └─► CC Switch
│
└─ 只想用一个 Agent + 一个官方订阅
    └─► 两者都不需要
```

### 4.2 典型用户画像

| 画像 | 推荐 | 理由 |
| :-- | :-- | :-- |
| **新手，只有 Claude Code + 一个中转站** | CC Switch | 三步搞定，无需理解网关 |
| **前端/后端工程师，用 Claude Code + Codex** | CC Switch | 聚合模式 + 双工具管理 |
| **重度用户，5+ 账号，关心额度** | magpie | 路由策略 + 额度配速 |
| **隐私敏感（读代码库、.env）** | magpie | 本地脱敏 |
| **家里有 NAS** | magpie | Docker + 远程共享 |
| **团队/多人共用一台开发机** | 两者都可 | CC Switch 多账号切卡 / magpie 网关密钥 |
| **DevOps / 脚本党** | magpie | CLI + daemon + OTLP |
| **只用终端（SSH 服务器）** | magpie | TUI / Web / CLI |
| **Windows 用户** | 两者都可 | CC Switch 需 WebView2；magpie 有原生安装 |
| **用冷门 Agent（Droid / Qoder / Cline…）** | magpie | 70+ Agent 覆盖 |

---

## 5. 组合使用：两者能一起用吗

**技术上可以，但需要小心。**

### 5.1 为什么会想组合

- CC Switch 的 **MCP / Skills / 会话管理**更好用；
- magpie 的**路由策略 / 脱敏 / 多账号**更强；
- 有人想「CC Switch 管配置，magpie 管转发」。

### 5.2 冲突点

| 冲突 | 说明 |
| :-- | :-- |
| **都改同一份配置文件** | CC Switch 写 `~/.claude/settings.json` 的 `ANTHROPIC_BASE_URL`；magpie 也写。**后写的覆盖先写的**。 |
| **都起本地服务** | CC Switch 用 `15721`，magpie 用 `3425`，端口不冲突，但**会形成两跳**。 |
| **状态互相覆盖** | 在 CC Switch 里切供应商会覆盖 magpie 写入的配置，反之亦然。 |
| **谁是真相** | 两者都认为自己是「配置的唯一管理者」。 |

### 5.3 可行的组合方式

**方式 A：CC Switch 只管扩展，magpie 管模型**

1. 用 magpie 配置所有 Agent 的模型（指向 `127.0.0.1:3425`）；
2. 用 CC Switch 管理 **MCP / Skills / 提示词**，但**不要在 CC Switch 里切换供应商**。

> ⚠️ 风险：CC Switch 打开供应商页时可能会重新写入配置。建议在 CC Switch 里把该 Agent **从侧栏隐藏**。

**方式 B：CC Switch 管 Claude Code，magpie 管其他 Agent**

- 各管各的工具，互不干扰；
- 适合「Claude Code 用 CC Switch 的聚合，Codex 用 magpie 的路由组」。

**方式 C：串联**

```
Claude Code → CC Switch 路由(15721) → magpie(3425) → 供应商
```

- 理论上可行（CC Switch 的上游填 `http://127.0.0.1:3425`）；
- **不推荐**：两层转换、两层故障转移、排障困难。

### 5.4 官方态度

- **CC Switch 文档**：未提及与 magpie 共存；
- **magpie 文档**：提到可以导入 CC Switch 的配置（「导入能读取你在 CC Switch、Claude Code、Codex 和 Alma 里配置的 provider」）。

> 这说明 **magpie 把 CC Switch 视为「迁移来源」**，而不是共存对象。

### 5.5 建议

> **选一个作为主，另一个卸载或隐藏。** 同时用两个「配置管理者」迟早会互相覆盖，排障成本远大于收益。

---

## 6. 从一方迁移到另一方

### 6.1 CC Switch → magpie

**magpie 有官方导入**：

1. 打开 magpie → **供应商** → **导入…**；
2. 它会读取 CC Switch 配置的 provider；
3. 勾选要导入的，确认；
4. 之后在 **Agent** 页重新给每个工具选模型；
5. 验证：`magpie ls` + `magpie provider test <id>`。

**注意**：

- 导入的是**一份拷贝**，之后两边不再同步；
- 订阅登录需要重新登录（导入不含 OAuth 凭据）；
- CC Switch 写入的配置会被 magpie 覆盖——建议先在 CC Switch 里把 Agent 恢复成官方配置。

### 6.2 magpie → CC Switch

**CC Switch 没有官方的 magpie 导入**。

手动步骤：

1. 在 magpie 里运行 `magpie providers`，导出 provider 列表；
2. 在 CC Switch 里逐个**添加供应商**（预设或自定义）；
3. 每个 Agent 重新选模型；
4. 在 magpie 里运行 `magpie <agent> default`，把配置恢复原样；
5. 再让 CC Switch 接管。

> 或者：CC Switch 支持**深度链接导入**，可以自己生成 `ccswitch://` 链接批量添加。

### 6.3 迁移检查清单

- [ ] 已记录所有 provider 的**名称、base URL、Key、模型 ID**
- [ ] 已记录每个 Agent 当前用的**模型**
- [ ] 已记录**路由组 / 聚合**的成员和策略
- [ ] 已备份原工具的配置文件
- [ ] 已导出旧工具的配置（`magpie backup` / CC Switch SQL 导出）
- [ ] 迁移后逐个 Agent 验证能正常对话
- [ ] 迁移后**卸载或停用**旧工具，避免互相覆盖

---

## 7. 各自的限制与坑

### 7.1 CC Switch 的已知限制

| 限制 | 说明 |
| :-- | :-- |
| **无官方 CLI** | 服务器/自动化场景要依赖社区版 |
| **无 Docker** | 不能跑在 NAS 上 |
| **无脱敏** | 密钥会原样发给厂商 |
| **无请求中间件** | 不能改写请求 |
| **路由无认证** | 本地路由没有密钥机制，不适合暴露到局域网 |
| **聚合不做故障转移** | 选中的供应商挂了要手动换 |
| **聚合不支持 Claude 官方订阅** | 需要回直连 |
| **Codex 聚合重启麻烦** | 要重启守护进程，会中断任务 |
| **旧预设不自动升级** | 需要重新添加 |
| **「正在使用的供应商」不能删** | 需先切走 |
| **配置文件损坏时拒绝写入** | 需手动修好 |
| **官方文档未说明遥测** | 隐私需自行核实 |

### 7.2 magpie 的已知限制

| 限制 | 说明 |
| :-- | :-- |
| **必须常驻** | 网关退出则所有 Agent 不可用 |
| **无聚合** | 不能在 `/model` 里同时看到多家模型（路由组只能整组选） |
| **不接受 PR** | 想要功能只能去 Discord |
| **配置概念多** | provider / agent / gateway / routing group / profile，新手门槛高 |
| **Claude 订阅依赖 Claude Code** | 不能卸载 Claude Code |
| **Docker 里登录订阅要手动粘回调 URL** | 厂商跳回 localhost，容器收不到 |
| **遥测默认开启** | 需要手动关 |
| **脱敏会增加延迟** | 每个请求都要扫描 |
| **本地模型查找要下载** | 39 MB 或 1.6 GB |

### 7.3 两者共同的坑

| 坑 | 说明 |
| :-- | :-- |
| **Agent 只在启动时读配置** | 改完要开新会话（Claude Code 除外，它支持热重载） |
| **Codex 模型列表在启动时生成** | 改了要重启 Codex 应用 |
| **「检测连通」不等于「能用」** | 只测地址可达，不验证 Key 和模型 |
| **官方订阅不能走本地路由** | 两者都是（Codex 的 OpenAI Official 例外） |
| **用订阅当供应商有封号风险** | 两者都有风险提示 |
| **中转站会看到你的 prompt 和 Key** | 只选信任的 |
| **多个工具同时管同一份配置会打架** | 见第 5 章 |

---

## 8. 版本与时效说明

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **本文基准版本** | v4.0.25（2026-11-02） | v0.1.2829（2026-11-03） |
| **迭代速度** | 极快（几乎每天有版本） | 极快 |
| **文档同步情况** | ⚠️ README 已更新到 12 工具，用户手册部分章节仍是 10 工具 | ✅ 文档与代码同步较好 |
| **破坏性变更** | 有过（v3.7.0 JSON → SQLite 迁移） | 有过（配置格式调整） |

> **建议**：动手前先看最新 Release Notes。两家的界面文案和菜单层级都可能在两个版本之间变化。

---

## 附录 A：端口与路径对照

### A.1 端口

| 用途 | CC Switch | magpie |
| :-- | :-- | :-- |
| **本地网关/路由** | `15721` | `3425` |
| **Web 界面** | 无 | `3430`（Docker 里 `magpie web`） |
| **开发模式** | 无 | `3426`（`DEV_ADDR`） |

### A.2 数据目录

| 项 | CC Switch | magpie |
| :-- | :-- | :-- |
| **主目录** | `~/.cc-switch/` | `~/.config/magpie/` |
| **Windows** | `C:\Users\<用户名>\.cc-switch` | `%USERPROFILE%\.config\magpie` |
| **数据库** | `cc-switch.db`（SQLite） | `usage.jsonl` + `routing/` 等 |
| **设置** | `settings.json` | `settings.json` |
| **供应商** | 数据库内 | `providers.json`（0600） |
| **登录凭据** | `*_auth.json` | `logins.json`（0600） |
| **备份** | `backups/`（明文 SQL） | 手动 `magpie backup`（加密） |
| **日志** | `logs/cc-switch.log` | `app.log` |

### A.3 各工具配置文件（两者都改这些）

| 工具 | 文件 |
| :-- | :-- |
| Claude Code | `~/.claude/settings.json` |
| Codex | `~/.codex/config.toml` |
| Gemini CLI | `~/.gemini/.env` + `settings.json` |
| Grok Build | `~/.grok/config.toml` |
| OpenCode | `~/.config/opencode/opencode.json` |
| Claude Desktop | `Claude-3p/configLibrary/*.json` |
| Pi | `~/.pi/agent/models.json` |
| MiniMax Code | `~/.minimax/config.yaml` |
| Kimi Code | `~/.kimi-code/config.toml` |

---

## 附录 B：术语对照

| 概念 | CC Switch | magpie |
| :-- | :-- | :-- |
| 模型来源 | 供应商（Provider） | Provider（供应商） |
| 使用模型的工具 | 应用 / 受管应用 | Agent |
| 本地转发服务 | 本地路由 | 网关（Gateway） |
| 配置快照 | 项目（Profile） | 方案（Profile） |
| 多模型组合 | 聚合（Aggregation） | 路由组（Routing group） |
| 协议适配 | 上游格式 | 协议转换 |
| 请求改写 | 整流器 | 网关中间件 / 整流 |
| 账号中心 | 授权中心 | 订阅 / 账号 |
| 扩展定义 | — | 资料库（Library） |
| 一键导入链接 | 深度链接 `ccswitch://` | 导入链接 `magpie://import` |
| 多机共享 | — | 远程 magpie（Remote magpie） |
| 模型别名 | 模型映射 | `magpie model id` |
| 占位密钥 | `PROXY_MANAGED` | （magpie 不写占位符，直接写网关地址） |

---

## 附录 C：命令与入口对照

| 操作 | CC Switch | magpie |
| :-- | :-- | :-- |
| **打开主界面** | 双击应用 | `magpie` |
| **只看托盘** | 关闭窗口（最小化到托盘） | `magpie tray` |
| **终端界面** | ❌ | `magpie tui` |
| **浏览器界面** | ❌ | `magpie web` |
| **只跑网关** | ❌ | `magpie serve` |
| **后台服务** | ❌ | `magpie daemon install` |
| **列出 Agent** | 界面左侧栏 | `magpie ls` |
| **列出供应商** | 界面供应商页 | `magpie providers` |
| **列出模型** | 界面模型选择器 | `magpie models` |
| **添加供应商** | ＋ 添加供应商 | `magpie provider add <id> <key>` |
| **测试供应商** | 卡片 ⋯ → 检测连通 | `magpie provider test <id>` |
| **切换模型** | 点卡片「切换」 | `magpie claude <provider>/<model>` |
| **恢复官方** | 选官方供应商 | `magpie <agent> default` |
| **保存方案** | Agent 页底部「＋ 保存当前」 | `magpie save <name>` |
| **应用方案** | 点方案 chip | `magpie use <name>` |
| **用量统计** | 侧栏「用量统计」 | `magpie usage 7d` |
| **额度** | 供应商卡片 | `magpie quota` |
| **日志** | `~/.cc-switch/logs/` | `magpie log -f` |
| **备份** | 设置 → 数据 → 导出 SQL | `magpie backup` |
| **恢复** | 设置 → 数据 → 导入 | `magpie restore <file>` |
| **更新** | 应用内检查更新 | `magpie update` |
| **卸载** | 系统卸载 + 删 `~/.cc-switch` | `install.sh --uninstall --purge` |

---

> **最后总结：**
>
> **CC Switch 是「配置的遥控器」**——它让你不用手改 JSON，顺便提供了一条够用的本地路由。**轻、可退出、桌面优先、扩展管理强。**
>
> **magpie 是「模型的交换机」**——所有请求必经它，配置只是它顺手做的事。**重、常驻、策略强、能上服务器、有脱敏。**
>
> 如果你只想要「换个 Key 就完事」，选 CC Switch；
> 如果你想要「一个能思考、能分配、能保护隐私的模型入口」，选 magpie。
>
> **不要两个一起用**——两个配置管理者迟早会互相覆盖。
