> **一句话：** CC Switch 是一个跨平台桌面应用，把 Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build、OpenCode 等十多个 AI 编程工具的**供应商配置、MCP、Skills、提示词、会话和用量**收进一个界面——换 API 供应商不用再手改 JSON / TOML / YAML。

---

## 关于本文

- **适用对象**：第一次接触 CC Switch，想给自己的 AI 编程工具换供应商、做多账号管理、或想统一管理 MCP / Skills / 提示词的人。
- **资料依据**：官方中文 README（`README_ZH.md`）、官方中文用户手册（`docs/user-manual/zh/` 全部章节）、官方使用攻略（`docs/guides/`）、官方更新日志（`CHANGELOG.md`）、Kimi Code 接入说明。
- **版本基准**：**v4.0.25**（2026-11-02 发布）。CC Switch 迭代很快，界面文案、菜单位置、默认参数都可能变化；遇到对不上时，以应用内实际显示和最新 Release Notes 为准。
- **一处已知的版本差异**：README 已列出 **12 个工具**（新增 Kimi Code、Oh My Pi），但用户手册 1.1 / 1.3 仍按旧的 **10 个工具** 编写。本文以 README 为准，涉及手册旧文案的地方会注明。
- **安全提醒**：本文涉及的所有「用订阅当供应商」「OAuth 反向代理」功能，都可能违反厂商服务条款、导致账号被限制。文中有专门的**风险提示**章节，请务必先读再做。

**官方资料：**

| 资料 | 地址 |
| :-- | :-- |
| 官网（唯一官方站点） | <https://ccswitch.io> |
| 下载页 | <https://ccswitch.io/download> |
| GitHub 仓库 | <https://github.com/farion1231/cc-switch> |
| Releases | <https://github.com/farion1231/cc-switch/releases> |
| 中文 README | <https://github.com/farion1231/cc-switch/blob/main/README_ZH.md> |
| 中文用户手册 | <https://github.com/farion1231/cc-switch/tree/main/docs/user-manual/zh> |
| 更新日志 | <https://github.com/farion1231/cc-switch/blob/main/CHANGELOG.md> |
| 深链生成器 | <https://ccswitch.io/deplink.html> |
| Issues | <https://github.com/farion1231/cc-switch/issues> |

> ⚠️ **只从 ccswitch.io、GitHub Releases 或源码仓库获取 CC Switch。** 任何要求付费、充值或索取登录凭据的「CC Switch」网站或客户端都不是官方渠道。

---

## 目录

- [0. 10 分钟最短路径](#0-10-分钟最短路径)
- [1. CC Switch 是什么](#1-cc-switch-是什么)
- [2. 核心概念](#2-核心概念)
- [3. 支持的十二个工具](#3-支持的十二个工具)
- [4. 安装](#4-安装)
- [5. 首次启动会发生什么](#5-首次启动会发生什么)
- [6. 界面导览](#6-界面导览)
- [7. 添加供应商](#7-添加供应商)
- [8. 切换供应商](#8-切换供应商)
- [9. 验证配置](#9-验证配置)
- [10. 本地路由：让 Claude Code 用 GPT、让 Codex 用 Claude](#10-本地路由让-claude-code-用-gpt让-codex-用-claude)
- [11. 故障转移与熔断](#11-故障转移与熔断)
- [12. 聚合模式](#12-聚合模式)
- [13. Claude Desktop 专项](#13-claude-desktop-专项)
- [14. Claude 官方多账号](#14-claude-官方多账号)
- [15. 授权中心与订阅当供应商](#15-授权中心与订阅当供应商)
- [16. MCP、提示词、Skills、会话、工作区](#16-mcp提示词skills会话工作区)
- [17. 用量、成本与额度](#17-用量成本与额度)
- [18. 项目：整套配置一键切换](#18-项目整套配置一键切换)
- [19. 数据、备份、云同步与配置文件](#19-数据备份云同步与配置文件)
- [20. 深度链接与配置分享](#20-深度链接与配置分享)
- [21. 环境变量冲突](#21-环境变量冲突)
- [22. 工具安装与升级](#22-工具安装与升级)
- [23. 设置项全表](#23-设置项全表)
- [24. 常见问题 FAQ](#24-常见问题-faq)
- [25. 故障排查手册](#25-故障排查手册)
- [26. 卸载](#26-卸载)
- [27. 术语表](#27-术语表)
- [附录 A：各工具配置文件速查](#附录-a各工具配置文件速查)
- [附录 B：默认参数速查](#附录-b默认参数速查)
- [附录 C：上线前检查清单](#附录-c上线前检查清单)

---

## 0. 10 分钟最短路径

如果你不想先读概念，照着做即可。

### 场景 A：我只有一个 API Key，想给 Claude Code 换上

1. 安装 CC Switch（见 [4. 安装](#4-安装)）。
2. 打开应用，左侧栏选中 **Claude Code**。
3. 点页面右上角的 **＋（添加供应商）**。
4. 在预设列表里搜厂商名（如 `deepseek`、`kimi`、`智谱`、`openrouter`），点它。
5. 填入 **API Key**，点页脚 **添加**。
6. 回到供应商列表，点新卡片的 **切换**。
7. **Claude Code 无需重启**，新开一个 `claude` 会话即可。

### 场景 B：我想在 Codex 里用别家的模型

1. 左侧栏选中 **Codex** → **＋ 添加供应商**。
2. 选预设（如 Kimi、DeepSeek、智谱 GLM），填 Key，添加。
3. 点卡片的 **切换**。
4. **关闭并重新打开终端**，再运行 `codex`。

> 如果预设卡片带 **「需要路由」** 徽章，说明上游协议和 Codex 不一致，要先按 [10. 本地路由](#10-本地路由让-claude-code-用-gpt让-codex-用-claude) 开启路由。

### 场景 C：我想让 Claude Code 用 GPT

→ 走 **本地路由 + Codex OAuth** 或 **本地路由 + Responses 网关**，见 [10. 本地路由](#10-本地路由让-claude-code-用-gpt让-codex-用-claude)。

### 最短验收

1. 卡片显示 **「使用中」**；
2. 重开对应工具，随便问一句能正常回复；
3. 侧栏 **用量统计** 里能看到这次请求（不开路由也能从会话记录统计到）。

三条都成立，第一步就完成了。

---

## 1. CC Switch 是什么

### 1.1 它解决的问题

Claude Code、Codex、Gemini CLI 等工具各有各的配置格式：JSON、TOML、YAML、`.env`。换一个 API 供应商，就得手动改配置文件；MCP、Skills 和提示词还要在每个工具里分别维护。

具体痛点：

| 痛点 | 表现 |
| :-- | :-- |
| **多供应商切换麻烦** | 官方、中转站、编程套餐来回换，每次都要改 JSON |
| **配置分散** | 每个工具一套格式，改错一个逗号整个工具起不来 |
| **订阅只能在一处用** | 买了 ChatGPT 订阅，却只能在 Codex 里用 |
| **无法监控用量** | 不知道调用了多少次、花了多少钱 |
| **服务不稳定** | 单一供应商挂了，工作流直接中断 |
| **扩展难管理** | MCP、Skills、提示词在每个工具里重复配置 |

CC Switch 把这些收进**一个桌面应用**：

- **一键切换供应商**：只替换地址、Key、模型等连接信息，插件、Hook、MCP、注释原样保留；
- **90+ 供应商预设**：选预设、填 Key 即可，也能自定义；
- **三种连接模式**：直连 / 路由 / 聚合；
- **统一管理扩展**：MCP、Skills、提示词按工具勾选同步；
- **会话可读**：把各工具的会话记录整理成阅读页；
- **用量与额度**：不开路由也能统计 Token 和花费。

### 1.2 它的技术形态

- **桌面应用**，基于 **Tauri 2 + Rust**，前端 **React 18 + TypeScript**；
- 数据存在本地 **SQLite**（`cc-switch.db`）+ 一个 JSON 设置文件；
- 支持 **Windows / macOS / Linux**；
- 有系统托盘，可快速切换供应商；
- **没有官方命令行版本**（服务器场景见 [1.5](#15-它不是什么)）。

### 1.3 三种连接模式

这是理解 CC Switch 的关键。Claude Code、Codex、Gemini CLI、Grok Build 的页面顶部都有 **「直连 / 路由 / 聚合」** 标签（聚合只有 Claude Code 和 Codex 有）。

| 模式 | 请求怎么走 | 能做什么 | 适合谁 |
| :-- | :-- | :-- | :-- |
| **直连** | 客户端直接连供应商 | 改配置文件里的关键字段 | 日常使用，一家供应商够用 |
| **路由** | 先经过本机 CC Switch（默认 `127.0.0.1:15721`）再转给供应商 | 接口格式转换、自动故障转移、热切换、逐条请求日志 | 想跨协议用模型、要高可用 |
| **聚合** | 同样经过本机路由，但按 `/model` 里选的模型分发 | 多家供应商的模型出现在同一个模型列表 | 想在客户端里随时换模型 |

**重要规则：**

- **点标签只是查看**，不会改配置；真正切换要点页面上写明后果的按钮：「开始路由」「切换到聚合模式」「回到直连」。
- **一个应用同一时间只能处于三种模式之一。**
- **路由和聚合都需要 CC Switch 保持运行。** 退出 CC Switch 时会先写回直连配置，下次启动再接上。
- **官方订阅不经过路由**（Codex 的 OpenAI Official 例外）。

### 1.4 切换式 vs 共存式

| 类型 | 工具 | 行为 | 按钮 |
| :-- | :-- | :-- | :-- |
| **切换式** | Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build | 同一时间只启用一个供应商，切换时改写该工具的配置文件 | 「切换 / 启用」 |
| **共存式** | OpenCode、OpenClaw、Hermes、Pi、MiniMax Code、Kimi Code、Oh My Pi | 多个供应商同时写进工具自身的配置，在工具里选 | 「添加 / 移除」（Pi、Oh My Pi 为「启用 / 移除」） |

**共存式工具不支持**：托盘切换、故障转移队列、本地路由、聚合。

### 1.5 它不是什么

- **不是模型提供商**：它不卖 token，也不转发到自己的服务器（路由是本机进程）。
- **不是 Agent**：它不会替你写代码，只负责配置管理和本机转发。
- **没有官方 CLI / 无界面版本**：官方只提供图形界面桌面版。服务器、SSH 远程场景推荐社区维护的 [CC Switch CLI](https://github.com/SaladDay/cc-switch-cli)（TUI + 命令行，支持 Claude Code、Codex、Gemini CLI、OpenCode、OpenClaw、Hermes、Pi）。
- **不会改你的其他配置**：只替换关键字段，插件、Hook、权限、MCP、注释和排版都保留。
- **不能删除正在使用的供应商**：这是有意设计——卸载 CC Switch 后工具仍能正常工作。

---

## 2. 核心概念

### 2.1 供应商（Provider）

一份「连接信息」：名称、请求地址、API Key、模型、上游接口格式。每个工具可以有多张供应商卡片。

卡片上可能有这些徽章：

| 徽章 | 含义 |
| :-- | :-- |
| **使用中 / 路由中 / 当前默认** | 当前生效的供应商 |
| **官方** | 厂商官方供应商（Claude Official、OpenAI Official 等） |
| **需要路由** | 上游协议与客户端不一致，必须开本地路由 |
| **直连 / 模型映射** | Claude Desktop 专用，表示接入方式 |

### 2.2 连接模式

见 [1.3](#13-三种连接模式)。

### 2.3 上游格式

告诉 CC Switch「这个供应商说的是哪种 API」：

| 格式 | 说明 | 谁需要 |
| :-- | :-- | :-- |
| **Anthropic Messages（原生）** | Claude 格式 | Claude Code、Claude Desktop |
| **OpenAI Chat Completions** | 大多数第三方网关 | 需要路由 |
| **OpenAI Responses API** | OpenAI 新接口，Codex 原生 | Codex 原生；Claude 需路由 |
| **Gemini Native generateContent** | Google 原生 | 需要路由 |

**选错格式的典型表现**：404 或 405。

### 2.4 项目（Profile）

把 Claude Code 或 Codex 当前的**供应商、MCP、Skills、提示词**保存成一个命名快照（Claude Desktop 只保存供应商）。之后一键整套切换；切到其他项目时，当前状态会自动存回原项目。

项目切换器默认隐藏，要在 **设置 → 通用 → 显示项目切换** 打开。

### 2.5 授权中心

统一管理第三方 OAuth 账号：

- GitHub Copilot
- ChatGPT（Codex OAuth）
- xAI（Grok）
- Kimi Code
- Claude 官方多账号（v4.0.21+）

把订阅当作供应商使用。

### 2.6 本地路由

CC Switch 在本机起的 HTTP 服务，默认 `127.0.0.1:15721`。它负责：

- 接口格式转换（Anthropic / OpenAI Chat / OpenAI Responses / Gemini Native 互转）；
- 故障转移；
- 热切换；
- 请求日志与用量。

### 2.7 数据目录

默认 `~/.cc-switch/`（Windows 为 `C:\Users\<用户名>\.cc-switch`）。

---

## 3. 支持的十二个工具

### 3.1 功能矩阵

| 工具 | 供应商 | 本地路由 | 聚合 | 托盘切换 | MCP | Skills | 提示词 | 会话 | 用量统计 |
| :-- | :-- | :--: | :--: | :--: | :--: | :--: | :-- | :--: | :--: |
| Claude Code | 切换 | ✓ | ✓ | ✓ | ✓ | ✓ | `CLAUDE.md` | ✓ | ✓ |
| Claude Desktop | 切换 | 模型映射时 | – | ✓ | – | – | – | – | 模型映射时 |
| Codex | 切换 | ✓ | ✓ | ✓ | ✓ | ✓ | `AGENTS.md` | ✓ | ✓ |
| Gemini CLI | 切换 | ✓ | – | ✓ | ✓ | ✓ | `GEMINI.md` | ✓ | ✓ |
| Grok Build | 切换 | ✓ | – | ✓ | ✓ | ✓ | `AGENTS.md` | ✓ | ✓ |
| OpenCode | 共存 | – | – | – | ✓ | ✓ | `AGENTS.md` | ✓ | ✓ |
| OpenClaw | 共存 | – | – | – | – | – | 工作区编辑器 | ✓ | – |
| Hermes | 共存 | – | – | – | ✓ | ✓ | 记忆管理 | ✓ | – |
| Pi | 共存 | – | – | – | ✓ | ✓ | `AGENTS.md`、`SYSTEM.md`、提示词模板 | ✓ | ✓ |
| MiniMax Code | 共存 | – | – | – | ✓ | ✓ | `AGENTS.md` | ✓ | ✓ |
| Kimi Code | 共存 | – | – | – | ✓ | ✓ | `AGENTS.md` | ✓ | ✓ |
| Oh My Pi | 共存 | – | – | – | ✓ | ✓ | `AGENTS.md` | ✓ | – |

**说明：**

- **切换**：同一时间只启用一个供应商；**共存**：多个供应商同时写入工具自身的配置，在工具里选择。
- **Claude Desktop 没有独立的 MCP / Skills / 提示词 / 会话**：这些面板操作的是 Claude Code 的数据。
- **OpenClaw、Hermes 暂不支持用量统计**。
- **Hermes 只显示最近 500 个会话**，且不支持命令行恢复。
- **Claude Code 的聚合模型需要 2.1.243 或更新版本**。

### 3.2 官方供应商

每个切换式工具都自带一个官方供应商：

| 工具 | 官方供应商 |
| :-- | :-- |
| Claude Code | Claude Official |
| Claude Desktop | Claude Desktop Official |
| Codex | OpenAI Official |
| Gemini CLI | Google Official |
| Grok Build | Grok Official |

**用途**：切过去，重启工具，按工具自己的登录流程操作（如 Claude Code 的 `/login`、Codex 的 `codex login`），就能恢复官方登录。删掉了可以从预设里重新添加。

---

## 4. 安装

### 4.1 系统要求

| 系统 | 最低版本 | 架构 |
| :-- | :-- | :-- |
| Windows | Windows 10 及以上 | x64 / ARM64 |
| macOS | macOS 12 (Monterey) 及以上 | Intel (x64) / Apple Silicon (arm64) |
| Linux | glibc 2.35+ 和 WebKitGTK 4.1 | x64 / ARM64 |

**Linux 支持的发行版**：Ubuntu 22.04+、Debian 12+、较新的 Fedora、Arch。
**不支持的**：RHEL / Rocky / Alma 8–9（系统库版本不够）。

> CC Switch **本身不需要 Node.js**。它管理的是你已经装好的 Claude Code 等工具。

### 4.2 Windows

**安装包：**

1. 从 [Releases](https://github.com/farion1231/cc-switch/releases) 或 [下载页](https://ccswitch.io/download) 下载 `CC-Switch-v{版本号}-Windows.msi`；
2. ARM 版 Windows 下载 `CC-Switch-v{版本号}-Windows-arm64.msi`；
3. 双击运行，按提示完成安装。

**绿色版：**

1. 下载 `CC-Switch-v{版本号}-Windows-Portable.zip`；
2. 解压到任意目录；
3. 运行 `CC-Switch.exe`。

**静默安装（跳过桌面快捷方式）：**

```powershell
msiexec /i "CC-Switch-v{版本号}-Windows.msi" /qn CREATEDESKTOPSHORTCUT=0
```

**常见问题：**

- 双击安装包没反应 → 右键 → 属性 → 常规 → 安全 → 勾选「解除锁定」；
- 安装后无法启动 → 装 [Microsoft Edge WebView2](https://developer.microsoft.com/en-us/microsoft-edge/webview2/)，并把 CC Switch 加入杀毒软件白名单。

### 4.3 macOS

**方式一：Homebrew（推荐）**

```bash
brew install --cask cc-switch
```

更新：

```bash
brew upgrade --cask cc-switch
```

**方式二：手动下载**

1. 下载 `CC-Switch-v{版本号}-macOS.dmg`（推荐）或 `.zip`；
2. 这是 **Universal 通用包**，Apple Silicon 和 Intel Mac 都能原生运行；
3. 打开 DMG，把 `CC Switch.app` 拖到「应用程序」。

> macOS 版本已通过 **Apple 代码签名和公证**，可直接打开，无需额外操作。

### 4.4 Linux

**Arch Linux：**

```bash
paru -S cc-switch-bin
# 或
yay -S cc-switch-bin
```

**Debian / Ubuntu：**

```bash
sudo dpkg -i CC-Switch-v{版本号}-Linux-*.deb
# 有依赖问题时
sudo apt-get install -f
```

**Fedora 等 RPM 发行版：**

```bash
sudo dnf install ./CC-Switch-v{版本号}-Linux-*.rpm
```

**AppImage（通用）：**

```bash
chmod +x CC-Switch-v{版本号}-Linux-*.AppImage
./CC-Switch-v{版本号}-Linux-*.AppImage
```

**AppImage 启动失败的排查：**

| 报错 | 处理 |
| :-- | :-- |
| `GLIBC_2.xx not found` | 系统版本过旧，不满足 glibc 2.35+ |
| FUSE 相关错误 | 安装 FUSE 2 兼容库（如 Ubuntu 的 `libfuse2`），或改用 `.deb` / `.rpm` |
| 托盘图标不显示 | 安装 `libappindicator` |

**Wayland + NVIDIA 点击无响应 / 缩放后黑屏：**

AppImage 会强制 `GDK_BACKEND=x11` 规避历史上的原生 Wayland 崩溃；但在较新的 Wayland + NVIDIA 环境下，这会导致网页内容区点不动。用内置开关切回原生 Wayland：

```bash
CC_SWITCH_GDK_BACKEND=wayland ./CC-Switch-*.AppImage
```

从桌面图标启动时，写进 `.desktop` 的 `Exec=` 行：

```
Exec=env CC_SWITCH_GDK_BACKEND=wayland /path/to/AppImage
```

> 该变量是通用的：在 tiling Wayland 合成器（sway/Hyprland）下若反而点击失效，可设 `CC_SWITCH_GDK_BACKEND=x11`。

**Flatpak**：官方 Release 不包含 Flatpak 包，可从 `.deb` 自行构建（见仓库 `flatpak/README.md`）。

### 4.5 验证安装

1. 应用窗口正常显示；
2. 系统托盘出现 CC Switch 图标；
3. 左侧栏能看到已显示的受管应用，点击能进入供应商页。

### 4.6 自动更新

- 启动时自动检查更新；
- 有新版本时界面显示提示；
- 点击即可下载安装；
- 也可以在 **设置 → 关于** 里手动检查；
- 便携版需要手动下载。

### 4.7 先装好要管理的工具

CC Switch 不替你装 Claude Code 等工具（除了可以在「应用」页一键装）。**推荐顺序**：先装 CC Switch，再用它里面的「应用」页安装工具；或者按各工具官方方式装好。

| 工具 | 安装命令（macOS / Linux） |
| :-- | :-- |
| Claude Code | `curl -fsSL https://claude.ai/install.sh \| bash` 或 `npm i -g @anthropic-ai/claude-code` |
| Codex | `npm i -g @openai/codex` 或 `brew install --cask codex` |
| Gemini CLI | `npm i -g @google/gemini-cli` |
| Grok Build | `curl -fsSL https://x.ai/cli/install.sh \| bash` 或 `npm i -g @xai-official/grok` |
| OpenCode | `curl -fsSL https://opencode.ai/install \| bash` 或 `npm i -g opencode-ai` |
| OpenClaw | `npm i -g openclaw` |
| Hermes | `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh \| bash` |
| Pi | `npm i -g @earendil-works/pi-coding-agent` |

> 用 npm 安装需要先装 Node.js。各工具要求不同（Claude Code 需要 22+，OpenClaw 需要 24.16+ 的 24.x 或 26.1+），建议装当前 LTS。
>
> 国内 npm 慢可以加 `--registry=https://registry.npmmirror.com`，或全局设置镜像源：
>
> ```bash
> npm config set registry https://registry.npmmirror.com
> ```

---

## 5. 首次启动会发生什么

第一次启动时，CC Switch 会**自动做三件事**：

1. **把 Claude Code、Codex、Gemini CLI、Grok Build 的现有配置导入**为名为 `default` 的供应商；
2. 为这几个工具和 Claude Desktop **添加官方供应商**；
3. 把每个工具的配置文件**第一次改写前**备份到 `~/.cc-switch/backups/live-first-write/`。

**原有配置不会丢失。**

> 如果你在 Claude Code 里已经配了很多供应商，之后还可以在 Claude Desktop 页点 **「将 Claude Code 中已有的供应商导入」** 一键带过去（见 [13.3](#133-从-claude-code-一键导入)）。

---

## 6. 界面导览

### 6.1 主界面布局

```
┌────────────┬──────────────────────────────────────────┐
│  左侧栏     │  内容区                                   │
│            │  ┌────────────────────────────────────┐  │
│  应用列表   │  │ 页头：应用名 · 添加供应商 · ⋯ 菜单   │  │
│  ├ Claude  │  ├────────────────────────────────────┤  │
│  ├ Codex   │  │ 直连 / 路由 / 聚合 标签             │  │
│  ├ Gemini  │  ├────────────────────────────────────┤  │
│  └ …       │  │ 供应商卡片列表                       │  │
│            │  │  ┌──────┐ ┌──────┐ ┌──────┐        │  │
│  全局页     │  │  │ 卡片 │ │ 卡片 │ │ 卡片 │        │  │
│  ├ MCP     │  │  └──────┘ └──────┘ └──────┘        │  │
│  ├ Skills  │  └────────────────────────────────────┘  │
│  ├ 提示词   │                                          │
│  ├ 会话     │                                          │
│  ├ 授权中心 │                                          │
│  └ 用量统计 │                                          │
│            │                                          │
│  底部       │                                          │
│  ├ 应用     │                                          │
│  └ 设置     │                                          │
└────────────┴──────────────────────────────────────────┘
```

### 6.2 左侧栏

| 区域 | 内容 |
| :-- | :-- |
| **应用列表** | 你选择显示的受管应用。点一个应用，右侧显示它的供应商页。应用名旁的小标记表示它当前用的是路由、聚合还是模型映射，出问题时会提示「需要处理」 |
| **全局页** | MCP、Skills、提示词、会话、授权中心、用量统计（有今日花费时会显示「今日 $xx」）。这些页面不属于某个应用，每个只有一个入口 |
| **底部** | 「应用」（安装、升级、选择侧栏显示哪些应用）和「设置」 |

- 点侧栏顶部的收起按钮，侧栏缩成只剩图标；
- 「应用」或「设置」旁出现圆点，表示有 CLI 工具或 CC Switch 本身有新版本。

### 6.3 应用页页头

页头右侧有：

| 元素 | 作用 |
| :-- | :-- |
| **添加供应商** | 添加新的供应商 |
| **项目切换** | 默认隐藏，需在设置里打开「显示项目切换」 |
| **⋯ 菜单** | 查看会话、配置目录、安装与升级、在侧栏隐藏、查看此应用的用量 |

Hermes 页头还有「Web UI」按钮。OpenClaw 和 Hermes 在页头下有一排页签：

| 应用 | 页签 |
| :-- | :-- |
| OpenClaw | 供应商、工作区、配置 |
| Hermes | 供应商、记忆 |

### 6.4 供应商卡片

从左到右：

| 元素 | 说明 |
| :-- | :-- |
| **拖拽手柄** | 悬停时出现，拖动调整顺序 |
| **供应商图标** | 品牌图标 |
| **供应商信息** | 名称、备注或端点地址，点击可打开官网；可能有「官方」「需要路由」等标签 |
| **用量信息** | 剩余额度或余额，多套餐时显示多行 |
| **状态 / 主操作** | 当前生效的显示「使用中」（路由模式下是「路由中」）；其他卡片是一个播放按钮 |
| **编辑按钮** | 编辑供应商配置 |
| **⋯ 更多** | 复制、检测连通、配置用量查询、打开终端（仅 Claude Code）、删除 |

**按钮在不同模式下的含义：**

| 当前模式 | 其他卡片的按钮 | 点下去 |
| :-- | :-- | :-- |
| 直连 | 「切换」 | 改配置文件，切到这一家 |
| 直连页但实际在路由/聚合 | 「回到直连并使用」 | 退出路由/聚合并使用这一家 |
| 路由 | 「路由到此」 | 只换路由目标，不改配置文件 |
| 聚合 | 「设为默认 / 添加」 | 设为默认供应商或加入聚合名单 |

**不能删除的供应商：**

- 切换式应用里**正在使用**的那一家（先切到别家）；
- 聚合模式下的**默认供应商**（先把别家设为默认）；
- 被 OpenClaw 默认模型引用的供应商；
- 被 MiniMax Code 的 `defaultModel` / `defaultLightModel` 引用的供应商；
- Hermes 当前使用中的供应商、由 Hermes 自己管理的只读供应商；
- Pi 读取到自身状态之前，删除按钮不可用。

### 6.5 系统托盘

右键托盘图标：

| 菜单项 | 功能 |
| :-- | :-- |
| **打开 CC Switch** | 显示主窗口并聚焦 |
| **应用子菜单** | Claude Code、Claude Desktop、Codex、Gemini CLI、Grok Build 各一个，标题显示「应用名 · 模式 · 在用供应商 · 额度摘要」；子菜单里点供应商即可切换 |
| **项目** | 切换项目（需开启「显示项目切换」） |
| **轻量模式** | 进入 / 退出仅托盘运行模式 |
| **打开官方网站** | 浏览器打开 ccswitch.io |
| **退出 CC Switch** | 完全退出；有应用在用路由或聚合时会提示「路由服务会停止」 |

**托盘子菜单的行为随当前模式变化：**

- **直连**：点供应商即切换；需要路由的后面标「（需要路由）」，点它会打开应用页并弹提示；
- **路由**：点供应商即换路由目标；官方订阅标「（官方订阅不经过路由）」，不能点；
- **路由 + 故障转移开启**：只读，列出队列顺序，要调整请到应用页；
- **聚合**：只列聚合名单里的供应商，点一家就是设为默认。

> **共存式应用不进托盘**，请在主界面操作。

### 6.6 轻量模式

| 特性 | 最小化到托盘 | 轻量模式 |
| :-- | :-- | :-- |
| UI 进程 | 保留在内存中 | **完全销毁** |
| 空闲资源占用 | 与正常运行相当 | 接近零 |
| 再次打开速度 | 瞬时 | 略慢（需要重建窗口） |
| 托盘切换功能 | 可用 | 可用 |
| 深链接唤起 | 可用 | 可用（按需重建） |

> 轻量模式状态**不持久**：下次正常启动会回到普通模式。

### 6.7 快捷键

| 快捷键 | 功能 |
| :-- | :-- |
| `Cmd/Ctrl + ,` | 打开设置 |
| `Cmd/Ctrl + F` | 搜索供应商（在供应商页） |
| `Esc` | 关闭搜索；在设置页返回主界面 |

### 6.8 设置目录

设置单独占一屏，左侧栏换成设置目录，分 6 组：

| 分组 | 内容 |
| :-- | :-- |
| **通用** | 外观、侧栏与页头、各应用的新版本、窗口与终端 |
| **应用配置** | 各应用的配置目录覆盖、各应用自己的选项 |
| **本地路由** | 服务、正在使用路由的应用、自动故障转移、整流器、全部退出路由 |
| **网络** | 全局出站代理 |
| **数据** | 存储位置、数据管理、备份与恢复、云同步、应用诊断日志 |
| **关于** | 版本信息、检查更新、GitHub、官网、发行说明 |

> 开关类设置**即时保存**；路径类设置（配置目录）改完要点该块的「保存」。

---

## 7. 添加供应商

### 7.1 打开添加页

1. 左侧栏选中要添加的应用；
2. 点页头右侧的 **添加供应商（＋）**。

添加页是整页界面，分两步：

1. **选预设**：挑一个预设，或选「自定义配置」；
2. **填写**：填 Key 等参数，点页脚「添加」。

> 如果表单动过（输入文字、勾选开关、换选项），这时离开当前页会先弹「有未保存的修改」确认。

### 7.2 用预设添加（推荐新手）

预设是预先配置好的供应商模板，通常只需填 API Key。

**第 1 步的布局：**

- **搜索框**：常驻顶部，可搜名称、网址或常见叫法（`kimi`、`智谱`、`openrouter`）；`⌘F` / `Ctrl+F` 回到搜索框；
- **左侧分类**：「全部」「账号登录」「模型厂商」「第三方平台」「云服务商」，OpenCode 另有「插件配置」；
- **右侧两列列表**：按显示名排序，「自定义配置」固定在「全部」最上面。

**分类说明：**

| 分类 | 内容 |
| :-- | :-- |
| **账号登录** | 不用 API Key，用已有账号登录：Claude 官方、ChatGPT（Codex）、GitHub Copilot、xAI、Google 官方。**除 Claude 以外都要经过路由** |
| **模型厂商** | 厂商自己的 API 和编程套餐。同一家的国内站、海外站、不同套餐合并成一行，选中后在下一步用「套餐」「地区」「版本」切换 |
| **第三方平台** | 聚合多家模型的平台，或转售官方模型的中转服务 |
| **云服务商** | 用云厂商账号调用模型，如 AWS Bedrock |
| **插件配置** | OpenCode 插件的模型分配配置 |

**操作步骤：**

1. 在搜索框或左侧分类里找到供应商，点击它；
2. 名称和端点会自动填充；
3. 填写 **API Key**；
4. （可选）填写备注；
5. 点页脚「添加」。

找不到想要的厂商时，搜索无结果的提示里有「用自定义配置添加」按钮。

**常见预设（部分）：** 智谱 GLM、MiniMax、DeepSeek、Kimi、PackyCode、OpenRouter、SiliconFlow、火山方舟、阿里云百炼、腾讯云、百度千帆、AWS Bedrock、NVIDIA NIM……

> 预设列表随版本更新，**以应用内实际显示为准**。

### 7.3 预设里要注意的三件事

**① 带「需要路由」徽章的预设**

说明上游协议和客户端不一致，要开启本地路由才能用。直连切换过去客户端会无法使用它。

**② 旧卡不会自动升级**

供应商保存的是**创建时的预设快照**。DeepSeek、GLM、Kimi 等预设已陆续改为原生 Responses；旧卡片要直连，建议**重新添加最新版预设**。

手动迁移时：把「上游格式」改为 Responses，并确认请求地址和模型映射与厂商的 Responses 端点匹配。

**③ 同一家不同套餐可能用不同协议**

例如 StepFun API 是 Responses，Step Plan 是 Chat Completions。以所选预设为准。

### 7.4 自定义配置

选择「自定义」后，需要手动填（或直接编辑 JSON / TOML）。

**Claude Code：**

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "your-api-key",
    "ANTHROPIC_BASE_URL": "https://api.example.com"
  }
}
```

| 字段 | 必填 | 说明 |
| :-- | :-- | :-- |
| `ANTHROPIC_API_KEY` | 是 | API 密钥 |
| `ANTHROPIC_BASE_URL` | 否 | 自定义端点地址 |
| `ANTHROPIC_AUTH_TOKEN` | 否 | 替代 API_KEY 的认证方式 |

**Codex：**

```toml
model_provider = "custom"
model = "gpt-5.6-sol"
model_reasoning_effort = "high"
disable_response_storage = true

[model_providers.custom]
name = "custom"
base_url = "https://api.example.com/v1"
wire_api = "responses"
requires_openai_auth = true
```

| 字段 | 必填 | 说明 |
| :-- | :-- | :-- |
| `model_provider` | 是 | 模型提供商名称（需与 `[model_providers.xxx]` 匹配） |
| `model` | 是 | 使用的模型 |
| `model_reasoning_effort` | 否 | 推理强度：`low` / `medium` / `high` 等 |
| `disable_response_storage` | 否 | 是否禁用响应存储 |
| `base_url` | 是 | API 端点地址 |
| `wire_api` | 否 | 固定为 `responses`；上游是其他格式时由「上游格式」和本地路由转换 |
| `requires_openai_auth` | 否 | 由预设设置，一般无需修改 |

> Codex 的第三方 API Key 会写进 `config.toml` 的 `experimental_bearer_token`，**不会**写进 `~/.codex/auth.json`。

**Gemini CLI：**

```json
{
  "env": {
    "GEMINI_API_KEY": "your-api-key",
    "GOOGLE_GEMINI_BASE_URL": "https://api.example.com"
  }
}
```

| 字段 | 必填 | 说明 |
| :-- | :-- | :-- |
| `GEMINI_API_KEY` | 是 | API 密钥 |
| `GOOGLE_GEMINI_BASE_URL` | 否 | 自定义端点地址 |
| `GEMINI_MODEL` | 否 | 指定模型 |

> 用 Vertex AI 的供应商可以用 `GOOGLE_API_KEY` 或 `GOOGLE_GENAI_USE_VERTEXAI` 代替 `GEMINI_API_KEY`。

### 7.5 自动获取模型列表

1. 确保已填 **API Key** 和 **端点地址**；
2. 点模型输入框旁的 **获取模型列表** 按钮（下载图标）；
3. CC Switch 用你的 Key 调 OpenAI 兼容的 `/v1/models` 端点；
4. 从分组下拉菜单里选模型。

**常见错误：**

| 错误 | 原因 |
| :-- | :-- |
| 401 / 403 | Key 不正确 |
| 404 / 405 | 供应商没有 `/v1/models` 端点，需手动填模型 ID |
| 解析失败 | 返回内容不符合 OpenAI 兼容格式 |
| 超时 | 端点慢或网络问题 |

### 7.6 统一供应商：一份配置给三个工具

适用于支持多种 API 格式的中转服务。

**创建：**

1. 在 Claude Code / Codex / Gemini CLI 的添加页里，选预设那一步的左侧分类最下面 **「跨应用」→「统一供应商」**；
2. 选一个预设类型；
3. 填名称、API Key、API 地址；
4. 勾选要同步的应用；
5. 保存。

**同步机制：**

- 修改统一供应商 → 所有关联应用同步更新；
- 删除统一供应商 → 关联的应用配置也会删除；
- 编辑时可以选「保存」（仅保存）或「保存并同步」（立即同步）；
- 也可以手动点卡片上的「同步」按钮。

### 7.7 导入已有配置

CC Switch 的供应商导入有两条路，**不要和 MCP 的「从应用导入」混淆**：

| 方式 | 入口 | 用途 |
| :-- | :-- | :-- |
| **深度链接导入** | 打开 `ccswitch://` 链接 | 单个供应商 / MCP / 提示词，来自他人分享或厂商按钮 |
| **数据库备份导入** | 设置 → 数据 → 数据管理 → SQL 导入导出 | 整份配置批量恢复（会**覆盖**现有数据库） |
| **从 Claude Code 导入**（仅 Claude Desktop） | Claude Desktop 页空列表里的「将 Claude Code 中已有的供应商导入」 | 把 Claude Code 那边的供应商一键带到 Claude Desktop |

> **注意**：CC Switch **没有**「从 Claude Code / Codex / OpenCode 自动扫描并导入全部供应商」的通用按钮。第一次启动时它只会把各工具**当前正在使用的那份配置**存成名为 `default` 的供应商；其余历史供应商需要手动添加，或用深度链接 / SQL 备份导入。
>
> MCP 页的「从应用导入」只针对 **MCP 服务器**，不涉及供应商。

**数据库备份导入步骤：**

1. 设置 → 数据 → 数据管理；
2. 点「选择 SQL 文件」（**只支持 CC Switch 自己导出的备份**）；
3. 选文件；
4. 点「导入」；
5. 确认覆盖现有配置。

> ⚠️ 导入会覆盖现有数据库，**建议先导出当前配置作为备份**。

### 7.8 深度链接导入

厂商或朋友可以给你一个 `ccswitch://` 链接：

```
ccswitch://v1/import?resource=provider&app=claude&name=My%20Provider&endpoint=https%3A%2F%2Fapi.example.com&apiKey=sk-xxx
```

打开后会弹出导入确认框，显示将添加的名称、地址、模型和 Key 去向；**确认后才保存**。

详见 [20. 深度链接](#20-深度链接与配置分享)。

### 7.9 高级选项

#### 上游格式（Claude）

| 格式 | 说明 | 是否需要路由 |
| :-- | :-- | :-- |
| **Anthropic Messages（原生）** | 原生 Anthropic API 格式，直连不转换 | 否 |
| **OpenAI Chat Completions** | 由本地路由转换 | 是 |
| **OpenAI Responses API** | 由本地路由转换 | 是 |
| **Gemini Native generateContent** | 由本地路由转换 | 是 |

#### 完整 URL 端点模式

默认情况下，CC Switch 把 `base_url` 当**前缀**，在后面拼 `/v1/chat/completions` 等固定路径。有些厂商要求非标准路径，这时打开请求地址旁的 **「完整 URL」** 开关，填完整端点。

| 模式 | `base_url` 填写 | 实际请求目标 |
| :-- | :-- | :-- |
| 默认（前缀拼接） | `https://api.example.com` | `https://api.example.com/v1/chat/completions` |
| **完整 URL 模式** | `https://api.example.com/custom/path/messages` | `https://api.example.com/custom/path/messages` |

> ⚠️ **完整 URL 模式必须配合本地路由使用。** 开启后卡片会显示「需要路由」。

#### Claude 快捷开关

添加 / 编辑 Claude 供应商时，JSON 编辑器上方有一组开关：

| 开关 | 效果 | 作用范围 |
| :-- | :-- | :-- |
| **隐藏 AI 署名** | 清除提交/PR 的署名元数据和会话链接（`attribution`） | 全局 |
| **启用 Tool Search** | 设置 `env.ENABLE_TOOL_SEARCH = "true"` | 跟随供应商 |
| **禁用自动升级** | 阻止 Claude Code 自动更新（`DISABLE_AUTOUPDATER=1`） | 全局 |
| **禁用 Artifact 工具** | 不把 Artifact 工具放进 tools 数组（部分第三方网关会因此返回 400） | 跟随供应商 |
| **关闭 auto mode 服务端检查** | 让 auto mode 直接用本机检查；新增第三方供应商时默认勾选，官方端点不要勾 | 跟随供应商 |

- **全局**：保存后写进 `~/.claude/settings.json`，对所有供应商生效；
- **跟随供应商**：保存在这个供应商里，切到它时写入，切走时移除；
- 取消勾选时，对应配置项会被**完全移除**。

#### Codex / Grok Build 的上游格式

| 选项 | 说明 |
| :-- | :-- |
| **Responses（原生）** | 上游原生支持 Responses API，直接连接 |
| **Chat Completions（需开启路由）** | 本地路由把 Responses 请求转成 Chat Completions，再把响应（含流式、推理、工具调用）转回 |
| **Anthropic Messages（需开启路由）** | 上游只提供 Anthropic Messages，由本地路由转换 |

#### 模型映射（仅 Codex）

Codex 非官方供应商的表单里有 **模型映射** 表：

| 列 | 说明 |
| :-- | :-- |
| 菜单显示名 | 在 `/model` 命令中显示的名称 |
| 实际请求模型 | 上游真实模型名，如 `deepseek-v4-flash` |
| 上下文窗口 | （可选）模型的上下文长度 |
| 思考等级 | （可选）该模型支持的思考等级 |

- 映射表会生成 Codex 的 `model_catalog_json`；
- 表中条目按填写内容原样保存，是模型列表的**唯一来源**；
- **默认模型**留空时，默认使用映射表第一行；
- **修改后需要重启 Codex** 才能刷新模型列表。

#### 思考能力

上游格式为 Chat Completions 时：

| 开关 | 含义 |
| :-- | :-- |
| **支持思考模式** | 上游支持开启/关闭 thinking（Kimi、GLM、Qwen 等通常属于这一类） |
| **支持思考等级** | 上游支持 low / high / max 等深度控制；启用后会自动启用思考模式 |

> ⚠️ 如果供应商只支持「思考模式」，在 Codex 里调思考等级**不会有任何效果**——CC Switch 不会把等级透传给这类上游。

#### Codex 1M 上下文窗口

添加 Codex 供应商时的开关：

- **启用**：设置 `model_context_window = 1000000`，并自动填 `model_auto_compact_token_limit = 900000`；
- **禁用**：移除这两个字段。

> 只在**新增**供应商时显示；编辑已有供应商时可通过高级配置直接调整。

#### 自定义图标

点名称左侧的图标打开选择器：

- **内置图标**：常见 AI 服务商和工具；
- **我的图标**：自己上传（PNG / JPG / WebP / SVG，不超过 2 MB）；
- 图片会缩成 96×96 的 PNG；
- 点「清除图标」退回名称首字母。

#### 端点测速

1. 编辑供应商，点请求地址旁的「管理与测速」；
2. 添加多个端点 URL；
3. 点「测速」；
4. 选延迟最低的。

**结果颜色：**

| 颜色 | 延迟 |
| :-- | :-- |
| 🟢 绿色 | < 300ms |
| 🟡 黄色 | 300–500ms |
| 🟠 橙色 | 500–800ms |
| 🔴 红色 | ≥ 800ms |

#### 备注与网站链接

- **备注**：账号用途（个人/工作）、套餐信息、到期时间；显示在卡片上，也支持搜索；
- **网站链接**：官网或控制台地址，卡片上可直接打开。

---

## 8. 切换供应商

### 8.1 主界面切换（直连模式）

1. 点击目标供应商卡片上的 **「切换」**；
2. CC Switch 更新配置文件；
3. 卡片按钮变成 **「使用中」**；
4. 按各应用的方式生效（见 [8.3](#83-生效方式)）。

### 8.2 托盘快速切换

1. 右键点击托盘图标；
2. 悬停到对应应用的子菜单；
3. 点击要切换到的供应商名称；
4. 托盘会短暂提示（如「已切换到 xxx，重启后生效」）。

### 8.3 生效方式

| 应用 | 生效方式 |
| :-- | :-- |
| **Claude Code** | ✅ **即时生效**（支持热重载，无需重启） |
| **Claude Desktop** | 需要**完全退出并重新打开** |
| **Codex** | 关闭并重新打开终端（桌面版要 ⌘Q 彻底退出） |
| **Gemini CLI** | 重启 Gemini CLI |
| **Grok Build** | 重启 Grok Build |
| **OpenCode / OpenClaw** | 关闭并重新打开终端 |
| **Hermes / Pi / MiniMax Code / Kimi Code** | 供应商添加后写入配置；在工具里选择模型 |

> **路由模式下**：切换供应商会立即作用于后续请求；但如果切换改变了模型，Codex、Gemini CLI、Grok Build 仍可能需要重启。
>
> **聚合模式下**：Claude Code 立即生效；Codex 需要彻底重启（见 [12.7](#127-什么时候需要重启)）。

### 8.4 切换时配置文件改了什么

CC Switch 只替换**关键字段**：

| 应用 | 文件 | 改什么 |
| :-- | :-- | :-- |
| Claude Code | `~/.claude/settings.json` | `env` 里的 `ANTHROPIC_*`、`AWS_*`、`CLAUDE_CODE_USE_BEDROCK`，以及 `model`、`apiKeyHelper` 等 |
| Codex | `~/.codex/config.toml` | `model_provider`、`model`、推理档位、`[model_providers.custom]` 表 |
| Gemini CLI | `~/.gemini/.env` + `settings.json` | `.env` 里的 `GEMINI_API_KEY`、`GEMINI_MODEL`、`GOOGLE_*`；`settings.json` 里只改 `security.auth.selectedType` 和 `model.name` |
| Grok Build | `~/.grok/config.toml` | `[models]` 下的 `default` 和 CC Switch 写入的那张 `[model."<名称>"]` 表 |
| Claude Desktop | 3P profile 配置 | 直连写地址和 Key；模型映射指向本机路由 |

**原样保留：**

- 插件、Hook、权限、MCP；
- 你自己加的环境变量；
- 注释和排版；
- `[mcp_servers]`、`[projects]`、你自己的其他供应商表。

> **第一次改写前会自动备份**到 `~/.cc-switch/backups/live-first-write/`，每个文件只备份一次。

### 8.5 在工具里改了模型，切走再切回来怎么又变回去了？

**模型属于关键字段，归供应商所有。**

- 在工具里换的模型（如 Claude Code 的 `/model`）会一直生效到下次切换；
- 切换时，配置文件里的模型会换成目标供应商保存的那个；
- CC Switch **不会**把你在工具里换的模型存回原来的供应商。

想长期用某个模型，请**在 CC Switch 里编辑这个供应商**。

### 8.6 共存式应用

| 应用 | 按钮 | 写入位置 | 额外操作 |
| :-- | :-- | :-- | :-- |
| OpenCode | 添加 / 移除 | `~/.config/opencode/opencode.json` | 在工具里选模型 |
| OpenClaw | 添加 / 移除 | `~/.openclaw/openclaw.json` | 卡片上「设为默认」设置默认模型 |
| Hermes | 添加 / 移除 | `~/.hermes/config.yaml` 的 `custom_providers` | 卡片上「启用」设为当前供应商 |
| Pi | 启用 / 移除 | `~/.pi/agent/models.json` | 在 Pi 里选模型 |
| MiniMax Code | 添加 / 移除 | `~/.minimax/config.yaml` 的 `custom_provider` | 在 MiniMax Code 里选模型 |
| Kimi Code | 添加 / 移除 | `~/.kimi-code/config.toml` 的 `providers`、`models` | 在 Kimi Code 的 `/model` 里选；也可用「切换」按钮 |
| Oh My Pi | 启用 / 移除 | `~/.omp/agent/models.yml`（供应商）+ `~/.omp/agent/config.yml`（默认模型） | 在工具里选模型 |

> 共存式应用**不支持**托盘切换和故障转移队列。

### 8.7 切换失败怎么办

| 原因 | 处理 |
| :-- | :-- |
| **配置文件被锁定** | 关闭正在运行的 CLI 工具，再切换 |
| **权限不足** | 检查配置目录的权限 |
| **配置格式错误** | 编辑供应商，检查 JSON / TOML 格式 |
| **配置文件本身有语法错误** | CC Switch 会提示「无法解析 …（第 N 行第 M 列）… 为避免覆盖你的配置，本次没有写入任何文件」，按位置修好再切 |

---

## 9. 验证配置

### 9.1 三步验证

1. **看卡片**：目标供应商显示 **「使用中」**；
2. **重启工具**（除 Claude Code 外）；
3. **发一句测试消息**：

```bash
# Claude Code
claude
> 你好，请简单介绍一下自己

# Codex
codex
> 你好，请简单介绍一下自己

# Gemini CLI
gemini
> 你好，请简单介绍一下自己

# OpenCode
opencode
> 你好，请简单介绍一下自己
```

### 9.2 更可靠的验证：看用量

打开左侧栏 **用量统计**：

- 不开路由时，数据来自各工具的**本地会话记录**；
- 开了路由时，还有**逐条请求日志**；
- 如果能看到刚才那次对话的 Token，说明配置真的生效了。

> 不要只问「你是什么模型」来判断——很多模型会答错自己的名字。

### 9.3 「检测连通」能证明什么

**只证明地址能连上。**

- 它向供应商的 base URL 发一个轻量请求；
- 收到任意 HTTP 响应（包括 401、403、404、5xx）都算「可达」；
- **不发送真实模型请求**，也不验证 API Key 和模型名。

> 「检测连通通过但请求失败」是正常现象，请检查 Key、模型名和上游格式。

---

## 10. 本地路由：让 Claude Code 用 GPT、让 Codex 用 Claude

### 10.1 为什么需要

Claude Code 说的是 **Anthropic Messages** 协议（`/v1/messages`）；GPT 系模型的上游说的是 **OpenAI Responses** 协议。两种协议的请求体、流式事件和返回结构完全不同，直接把地址填进 Claude Code 配置里，上游收到它不认识的请求，只能失败。

CC Switch 的做法：

1. 让 Claude Code 始终连**本机路由**，仍以 Anthropic Messages 发送；
2. 路由识别当前供应商是 Responses 格式后，把请求转成 Responses 发给上游；
3. 上游返回后，再把响应转回 Messages 形态给 Claude Code。

**工具调用、图片、PDF、思考配置都在转换范围内。**

### 10.2 支持路由的应用

**Claude Code、Codex、Gemini CLI、Grok Build**；Claude Desktop 的「模型映射」模式也经本地路由转发。

### 10.3 启动路由

**方式一：从应用页**

1. 左侧栏选中应用；
2. 点顶部 **「路由」** 标签（只是查看）；
3. 点 **「开始路由」**；
4. 在确认框里选择「路由到」哪家供应商；
5. 点「开始路由」确认。

确认框会写明后果：会改写客户端配置里的地址和密钥，真实密钥留在 CC Switch 里，其他设置不动。

**方式二：从设置页**

**设置 → 本地路由 → 服务 → 启动**。

> 路由服务一般**不需要手动启动**：点「开始路由」时，服务没在运行会自动启动。

### 10.4 路由配置

**设置 → 本地路由 → 服务：**

| 配置项 | 说明 | 默认值 |
| :-- | :-- | :-- |
| **监听地址** | 绑定的 IP，支持 IPv4、IPv6 和 `localhost` | `127.0.0.1` |
| **监听端口** | 1024–65535 | `15721` |
| **记录请求用量** | 把路由请求的用量与状态写入统计数据库 | 开启 |

修改后点 **「保存并重启服务」**。

**监听地址选择：**

| 地址 | 说明 |
| :-- | :-- |
| `127.0.0.1` | 仅本机可访问（推荐） |
| `0.0.0.0` | 允许局域网访问 |

**连接地址**：CC Switch 写进客户端配置的地址默认按监听地址推导（`0.0.0.0` 写成 `127.0.0.1`）。客户端无法通过本机回环访问时，在「连接地址」里填它能访问到的 IP 或主机名。

> **WSL2 场景**：默认 NAT 模式下的 Claude Desktop 访问不到 Windows 的 `127.0.0.1`。把监听地址设为 `0.0.0.0`，连接地址填 WSL 能访问到的 Windows IP（如 `vEthernet (WSL)` 网卡的地址），并在 Windows 防火墙放行端口。

### 10.5 路由时配置文件变成什么样

**Claude Code：**

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://127.0.0.1:15721"
  }
}
```

**Codex：**

```toml
[model_providers.custom]
base_url = "http://127.0.0.1:15721/v1"
```

**Gemini CLI：**

```bash
GOOGLE_GEMINI_BASE_URL=http://127.0.0.1:15721
```

**Grok Build：**

```
http://127.0.0.1:15721/grokbuild/v1
```

**Claude Desktop（模型映射）：**

```
http://127.0.0.1:15721/claude-desktop
```

**共同点：**

- API Key 替换为占位符 **`PROXY_MANAGED`**；
- 真实的供应商地址、Key 和模型都保存在 CC Switch 里；
- Claude Code 的模型名还会写成 `claude-sonnet-5` 之类的固定别名（`/model` 菜单里仍显示真实模型名）。

### 10.6 接口格式转换

**Claude Code / Claude Desktop 侧**（客户端发 Anthropic Messages）：

| 供应商上游格式 | 路由行为 |
| :-- | :-- |
| Anthropic Messages | 透传（不转换） |
| OpenAI Chat Completions | 转成 OpenAI Chat，响应反向转换 |
| OpenAI Responses API | 转成 OpenAI Responses，响应反向转换 |
| Gemini Native generateContent | 转成 Gemini 原生，响应反向转换 |

**Codex / Grok Build 侧**（客户端发 OpenAI Responses）：

| 供应商上游格式 | 路由行为 |
| :-- | :-- |
| Responses（原生） | 透传 |
| Chat Completions | 转成 Chat Completions，响应反向转换 |
| Anthropic Messages | 转成 Anthropic Messages，响应反向转换 |

### 10.7 停止路由

- **单个应用**：在应用页点 **「回到直连」**；
- **全部应用**：**设置 → 本地路由 → 全部回到直连**，确认后所有应用写回直连配置，然后停止路由服务。

> **有应用在用路由时不能停止服务**：「停止」按钮是灰的，会提示「还有应用在用路由服务，先让它们回到直连」。
>
> 退出路由时，**路由目标、故障转移队列和聚合名单都会保留**。

**退出后的处理：**

1. 把应用的配置文件写回直连供应商（进入路由前在用、卡片上标「直连」的那个）；
2. 保存请求日志；
3. 关闭连接。

> 退出 CC Switch 时也会先写回直连，应用的模式保持不变，下次启动再重新接上；如果没能接上，应用会回到直连并给出提示。

### 10.8 实战：在 Claude Code 里用 GPT

#### 方式一：第三方 Responses 网关（API Key）

**准备工作：**

- CC Switch 3.17.0 及以上；
- 已安装 Claude Code，并至少运行过一次；
- 一个兼容 OpenAI Responses API 的端点 + Key（**是 Responses，不是 Chat Completions**）。

**步骤：**

1. 左侧栏选 **Claude Code** → **＋ 添加供应商**；
2. 保持默认的 **自定义配置**；
3. 填写：
   - **供应商名称**：随意，如 `GPT Gateway`；
   - **API Key**：网关 Key（真实 Key 只保存在 CC Switch 里）；
   - **请求地址**：网关根地址，如 `https://gpt-gateway.example.com`，**不要以斜杠结尾**；
4. 展开 **高级选项**：
   - **API 格式**：改成 **`OpenAI Responses API（需开启路由）`**；网关只提供 Chat Completions 时选 `OpenAI Chat Completions（需开启路由）`；
   - **认证字段**：保持默认的 `ANTHROPIC_AUTH_TOKEN（默认）`，路由会以 `Authorization: Bearer <key>` 发给上游；除非网关文档明确要求 `x-api-key`，否则不要改（改错典型表现是 401/403）；
   - **模型映射**：**至少要填「默认兜底模型」**（如 `gpt-5.6`，以网关文档为准）；更精细的做法是按行指定 Sonnet / Opus / Haiku；
   - **声明支持 1M**：仅当网关确实以 100 万 token 及以上窗口服务该模型时才勾选；
5. 保存后，卡片上会出现 **「需要路由」** 标记。

**开启路由：**

1. 点顶部 **「路由」** 标签 → **「开始路由」**；
2. 选择这家供应商；
3. 如果 Claude Code 正在运行，**重开一个终端会话**（live 配置是启动时读取的）。

**验证：**

- 新开会话，用 `/model` 查看模型菜单，应显示映射里的显示名；
- 发一个小问题，看 **设置 → 本地路由** 里的「总请求数」增长；
- 用量看板里按上游真实模型显示。

#### 方式二：ChatGPT 订阅（Codex OAuth）

**准备工作：** 一个 ChatGPT Plus / Pro 订阅账号。

**步骤：**

1. 左侧栏选 **Claude Code** → **＋ 添加供应商**；
2. 在预设列表里选带 OpenAI 图标的 **`Codex`** 预设（它出现在 Claude Code 标签下是正确的）；
3. **不需要 API Key，也不需要填地址**——请求固定发往 ChatGPT 的 Codex 服务；
4. 点 **「使用 ChatGPT 登录」**：
   - 这是**设备码流程**；
   - CC Switch 会自动打开浏览器并把验证码复制到剪贴板；
   - 在浏览器页面粘贴验证码完成授权；
   - 应用内显示「等待授权中...」；
5. 登录成功后，「认证状态」显示已登录账号（邮箱）；
6. 支持多账号：可「添加其他账号」「设为默认」，或在这个供应商上指定使用某个账号。

**预填的模型档位：**

| 角色 | 默认模型 |
| :-- | :-- |
| 主模型 | `gpt-5.6-sol` |
| Sonnet | `gpt-5.6-sol` |
| Opus | `gpt-5.6-sol` |
| Haiku | `gpt-5.6-luna` |

预设还会把 `CLAUDE_CODE_MAX_CONTEXT_TOKENS` 和 `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 设为 `372000`。

**FAST 模式**：可选开关，开启后请求携带 `service_tier="priority"` 换取更低延迟，但会按更高速率消耗 ChatGPT 配额，**默认保持关闭**。

**凭据位置**：`~/.cc-switch/codex_oauth_auth.json`（**不是** `~/.codex/`），与 Codex CLI 自己的登录互不影响；Token 会在到期前自动刷新。

**能力边界与已知限制：**

| 项 | 说明 |
| :-- | :-- |
| **提示缓存** | 路由会为每个会话注入稳定的 `prompt_cache_key`，长对话不会每轮全价重发 |
| **思考** | Claude Code 的思考开关与等级会折算成 GPT 的 `reasoning.effort`（low/medium/high）；推理内容以加密形态跨轮完整回放 |
| **工具与多模态** | 多轮工具调用、图片与 PDF 输入都被完整转换 |
| **上下文窗口** | Claude Code 对路由供应商按默认 **200K** 窗口做自动压缩；超出 200K 的部分当前不会被用到。方式二上游上限是 372K、够不到 1M，**不要勾 1M** |
| **输出上限** | 方式二由 ChatGPT 服务端控制（`max_tokens` 不会下发）；方式一原样透传 |
| **联网搜索** | 取决于上游是否支持托管 `web_search` 工具 |
| **用量金额** | Token 计数准确，美元金额是按公开 API 价折算的**估算**，仅供参考 |

### 10.9 实战：在 Codex 里用 Claude

反向同理：

1. Codex 添加供应商，**上游格式** 选 **Anthropic Messages（需开启路由）**；
2. 开启路由并选它；
3. 重启 Codex。

详见官方攻略 `docs/guides/codex-claude-routing-guide-zh.md`。

### 10.10 整流器

**设置 → 本地路由 → 整流器**：自动修复部分上游不兼容的请求，例如：

- Thinking 签名不兼容时移除相关内容后重试；
- 上游不支持图片时降级。

---

## 11. 故障转移与熔断

### 11.1 前提条件

1. ✅ 应用已进入**路由模式**（聚合模式不做故障转移）；
2. ✅ 在「路由」标签里打开**故障转移开关**；
3. ✅ 队列里有供应商。

> 故障转移只支持 **Claude Code、Codex、Gemini CLI、Grok Build** 四个应用。官方账号不进队列（它用客户端自己的登录，不能和别家轮换）。

### 11.2 开启

1. 应用页切到 **「路由」** 标签，确认已处于路由模式；
2. 打开标签栏右侧带「故障转移」提示的开关（两条交叉的箭头图标）；
3. 首次开启会弹确认框，点「我已了解，继续启用」。

| 状态 | 行为 |
| :-- | :-- |
| 关闭 | 只路由到选定的一家供应商，不自动切换 |
| 开启 | 立即切换到队列 P1，并在请求失败时自动切换到下一个 |

### 11.3 管理队列

- **队列 · 按优先级**：请求先发给第一位（P1），连续失败时自动换下一位；
- **不在队列**：还没加入的供应商。

**操作：**

- 添加：在「不在队列」里点卡片上的 **「加入队列」**；
- 调整顺序：卡片上的上移、下移按钮，或直接拖动；
- 移除：点队列里卡片上的 **「移出队列」**。

> 队列顺序与首页供应商列表的顺序一致。

### 11.4 熔断器默认参数

| 配置 | 说明 | 通用默认 | Claude 默认 | 范围 |
| :-- | :-- | :--: | :--: | :-- |
| **失败阈值** | 连续失败多少次触发熔断 | 4 | 8 | 1–20 |
| **恢复成功阈值** | 半开状态下成功多少次后关闭熔断器 | 2 | 3 | 1–10 |
| **恢复等待时间** | 熔断后多久尝试恢复（秒） | 60 | 90 | 0–300 |
| **错误率阈值** | 错误率超过此值时打开熔断器 | 60% | 70% | 0–100% |
| **最小请求数** | 计算错误率前的最小请求数 | 10 | 15 | 5–100 |

> Claude 由于请求耗时较长，默认配置更宽松。

### 11.5 超时与重试

| 配置 | 说明 | 通用默认 | Claude 默认 | 范围 |
| :-- | :-- | :--: | :--: | :-- |
| **流式首字节超时** | 等待首个数据块的最大时间（秒） | 60 | 90 | 1–120 |
| **流式静默超时** | 数据块之间的最大间隔（秒） | 120 | 180 | 60–600（0 禁用） |
| **非流式超时** | 非流式请求的总超时（秒） | 600 | 600 | 60–1200 |
| **最大重试次数** | 请求失败时的重试次数 | 3 | 6 | 0–10 |

> Gemini 的默认最大重试次数为 **5**。

### 11.6 熔断状态

| 状态 | 说明 |
| :-- | :-- |
| **关闭** | 正常，允许请求 |
| **开启** | 熔断，跳过此供应商 |
| **半开** | 尝试恢复，发送试探请求 |

状态流转：`关闭 →（失败次数 ≥ 阈值）→ 开启 →（熔断时长到期）→ 半开 →（试探成功 ≥ 恢复阈值）→ 关闭`；半开试探失败则回到开启。

### 11.7 健康状态徽章

| 徽章 | 状态 | 说明 |
| :-- | :-- | :-- |
| 🟢 | 正常 | 连续失败 0 次 |
| 🟡 | 降级 | 有失败但未触发熔断 |
| 🔴 | 熔断 | 已触发熔断，暂时跳过 |

### 11.8 队列配置建议

1. **主供应商**：最稳定、最快的；
2. **第一备用**：次优；
3. **第二备用**：保底。

| 场景 | 失败阈值 | 熔断时长 |
| :-- | :-- | :-- |
| 高可用要求 | 2 | 30 秒 |
| 一般场景 | 3 | 60 秒 |
| 容忍偶发失败 | 5 | 120 秒 |

---

## 12. 聚合模式

### 12.1 它是什么

把**多家供应商的模型放进 Claude Code 或 Codex 的同一个模型列表**。在客户端的 `/model` 里选哪个模型，请求就发给哪一家，不用回 CC Switch 切换供应商。

**只支持 Claude Code 和 Codex**，两个应用的名单要分别配置。

### 12.2 与路由的区别

| | 路由 | 聚合 |
| :-- | :-- | :-- |
| 同一时间使用的供应商 | 一家（或按故障转移队列轮换） | 多家，按所选模型分发 |
| 故障转移 | 支持 | **不支持** |
| 切换供应商的方式 | 在 CC Switch 里切换 | 在客户端 `/model` 里选模型 |
| 适用应用 | Claude Code / Codex / Gemini CLI / Grok Build | 仅 Claude Code / Codex |

> 两者都经过本机路由服务，都需要 CC Switch 保持运行。一个应用同一时间只能是三种模式之一。

### 12.3 进入聚合模式

1. 左侧栏选 Claude Code 或 Codex → 点顶部 **「聚合」** 标签（只是查看）；
2. 配好各家的模型列表，把需要的供应商加入聚合名单；
3. 确认默认供应商；
4. 点 **「切换到聚合模式」**；
5. 在确认框里查看默认供应商和将出现在 `/model` 里的模型，确认。

切换后页面显示「聚合生效中 · 默认」，以及默认供应商和另外几家共多少个模型。

### 12.4 添加和移除供应商

- **添加**：在「可添加」列表里点卡片上的 **「添加」**，这家的模型就会出现在客户端的模型列表里；
- **移除**：在「已添加」列表里点 **「移除」**。

> ⚠️ **默认供应商不能移除**，请先把别家设为默认。

### 12.5 设置每家的模型

**配置模型列表和把供应商加入聚合名单是两个步骤**：保存供应商后，还要在「可添加」区域点「添加」。

#### 获取模型列表

1. 填好 API 地址、密钥或完成账号配置；
2. 点 **「获取模型列表」**；
3. 搜索并勾选需要的模型；
4. 点 **「添加所选模型」** 批量加入。

> 不必把返回的全部模型都加入，只留常用的几个即可。

#### 自动补全模型信息

加入列表后，CC Switch 会自动从 **models.dev** 获取模型资料，补全可匹配到的：

- 上下文窗口；
- 思考等级；
- 文字 / 图片输入能力。

> 自动补全只填写空缺项，保留已有参数，并提示信息来源。不同供应商部署的同名模型可能有不同能力，**以当前供应商实际支持的参数为准**。

#### 手动添加模型

供应商未开放模型列表接口、列表暂未包含新模型、或需要自定义别名时，点 **「手动添加」**，填供应商实际支持的模型 ID。

#### 默认模型

- 配置的模型会出现在客户端的模型选择器里；
- **★** 表示这家供应商的默认模型；
- Claude Code 中它位于列表第一项，供默认供应商的启动和后台任务使用；
- Codex 使用配置中指定的默认模型。

### 12.6 默认供应商与 `ccs-` 前缀

**默认供应商**承接所有不带前缀的请求。

- 在供应商卡片上点 **「设为默认」** 更换；
- 使用 Codex 官方账号时，可以把 **OpenAI Official 设为默认供应商**，再添加其他供应商；
- **OpenAI Official 只能作为默认供应商**，不能作为普通聚合成员加入。

**`ccs-` 前缀：**

- 除默认供应商外，其他已添加的供应商，模型会以 **`ccs-` 前缀**出现在 `/model` 里；
- 选择带 `ccs-` 前缀的模型 → 请求发给对应那一家；
- 选择不带前缀的 → 请求发给默认供应商。

> ⚠️ **聚合模式暂不支持 Claude 官方订阅。** 需要使用该订阅时，请回到直连模式，在 Claude Code 中通过官方登录流程使用。

### 12.7 什么时候需要重启

| 应用 | 聚合模型列表变化后 |
| :-- | :-- |
| **Claude Code** | **立即生效**（需要 2.1.243 或更新版本） |
| **Codex** | **需要彻底重启** |

**Codex 的重启方式：**

| 使用方式 | 操作方法 |
| :-- | :-- |
| **Codex CLI** | 只关掉终端重开 `codex` **不够**——所有终端里的 `codex` 共用一个后台守护进程。点 CC Switch 页面顶部提示里的 **「重启 Codex 守护进程」**（会中断正在运行的任务） |
| **Codex 桌面版** | 彻底退出再重开。macOS 按 `⌘Q`，只关窗口不行 |
| **编辑器插件** | 重开编辑器窗口 |

> 在已经加载好的模型列表里选择另一个模型，**不需要**每次重启。

### 12.8 Codex 子 agent 用另一个模型

聚合生效后，在聚合页签右上角点齿轮打开 **「聚合设置」**：

| 设置 | 说明 |
| :-- | :-- |
| **子代理模型** | 从聚合里已发布的模型中选一个；选「跟随主模型」恢复默认 |
| **子代理推理强度** | 只列所选模型支持的档位；选「模型默认」时用该模型自己的默认档 |
| **写在哪里** | `~/.codex/config.toml` 的 `[agents]`（`default_subagent_model`、`default_subagent_reasoning_effort`） |

- **不用重启 Codex**：新开的对话就会用上，已经开着的对话仍按原设置；
- **这是本机设置**，不跟供应商走；
- **模型不在聚合里了**：不写这两项，子 agent 暂时跟随主模型；加回来后自动恢复；
- **跨供应商**：子 agent 和主 agent 不在同一家时，必须打开「聚合模式下子 agent 用经典工具」（默认已打开）。

### 12.9 Codex 子 agent 读不到任务

**症状：**

- `Encrypted function output content could not be decrypted`
- `stream disconnected before completion`
- `unreadable_encrypted_agent_task`

**原因：** Codex 新版的子 agent 工具会让主 agent 那家供应商加密任务，换一家就解不开。

**解决：**

1. 在聚合页签的「聚合设置」里重新打开 **「聚合模式下子 agent 用经典工具」**（默认打开）；
2. **重启 Codex**。

**注意：**

- **有代价**：没有 `followup_task`、`interrupt_agent` 等新版功能；
- **只影响聚合模式**；
- **开了不生效**：如果 `~/.codex/config.toml` 的 `[features]` 里有 `multi_agent_v2 = true`，它的优先级高于这个开关，需要先删掉。

### 12.10 Codex 自动审批用另一个模型

Codex 开了自动审批（`approvals_reviewer = "auto_review"`）时，可以在「聚合设置」里选 **「自动审批模型」**：

- **默认**：默认那家是官方时用官方的 Auto Review（`codex-auto-review`）；是第三方时由主模型自己审批；
- **能选什么**：聚合里已发布的模型（默认那家自己的除外）；
- **写在哪里**：`cc-switch-model-catalog.json` 每一行的 `auto_review_model_override`；
- **改完要重启 Codex**；
- **选听指令的模型**：审批结果要按约定的 JSON 返回，经 Chat / Anthropic 格式转换的模型只靠提示词约束，回答不合格式会算作审批失败。

### 12.11 官方额度用完后第三方也发不出去

**症状：** 官方 5 小时额度用完后，Codex 锁住发送键，或模型选择器只剩「Luna Reserve」。

**原因：** 这是 Codex 客户端自己的限制——只要本机有 ChatGPT 官方登录，它就自己查询额度，发现用完就在请求发出前换掉模型，请求到不了 CC Switch。

**绕开：**

1. 在应用页顶部切到 **「直连」**；
2. 确认 **设置 → 应用配置 → Codex** 里的 **「非接管切换时保留官方登录」是关闭的**（默认关闭）；
3. 切换到一家第三方供应商（CC Switch 会把官方登录暂时收起来）；
4. 彻底重启 Codex，新开会话。

**恢复：** 官方额度恢复后切回「聚合」，再重启一次 Codex，收起来的官方登录会自动还回去。

> 代价：期间只能用这一家第三方的模型。

### 12.12 聚合的限制

- **不做故障转移**：队列和设置会保留，回到路由模式后恢复；
- **Codex 的 OpenAI Official 只能作默认供应商**；
- **暂不支持 Claude 官方订阅**；
- **默认供应商不能移除**；
- **需要 CC Switch 保持运行**；
- **回到路由模式时，聚合的模型会从 `/model` 撤下**（名单保留），已选中的聚合模型需要改回普通模型；
- **同一会话里换模型后，新模型要重新建立提示词缓存，第一轮费用会高一些**。

---


---

> **本文篇幅较长，GitHub Issue 单篇上限 65536 字符，因此分为两篇：**
> - 第 1 篇（本篇）：第 0–12 章 —— 概念、安装、供应商、切换、验证、本地路由、故障转移、聚合
> - 第 2 篇：第 13–27 章 + 附录 —— Claude Desktop、Claude 多账号、授权中心、MCP/Skills、用量、数据备份、FAQ、排障、卸载、术语、附录
