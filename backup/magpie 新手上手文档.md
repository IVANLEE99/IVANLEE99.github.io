> **一句话：** magpie 是一个跑在你电脑上的本地应用 + 本地网关，让你所有的 AI 编程 Agent（Claude Code、Codex、Gemini CLI、OpenCode……）共用一套模型配置——任意 Agent 用任意模型，额度用完自动换账号。

---

## 关于本文

- **适用对象**：第一次接触 magpie、想把自己的 Claude Code / Codex / Gemini CLI / OpenCode 接上不同模型的人。
- **资料依据**：官方中文 README、官方中文《快速上手》、官方英文参考文档（`docs/reference.md`）、官方中文 Docker / 远程 magpie / 本地脱敏 / 隐私文档。
- **时效说明**：magpie 迭代很快，界面文案、模型 ID、命令参数都可能变化。本文写作时间为 **2026 年 11 月**；遇到对不上的地方，以应用内界面、`magpie help` 和官方文档为准。
- **模型名说明**：文中的 `deepseek/deepseek-v4-pro`、`moonshot/kimi-k2.5` 等只是示例，实际可选模型以你添加 provider 后 `magpie models` 的输出为准。

**官方资料：**

| 资料 | 地址 |
| :-- | :-- |
| 官网 / 下载 | <https://usemagpie.ai/zh/> |
| 中文快速上手 | <https://usemagpie.ai/docs/zh/start> |
| 完整参考（英文） | <https://github.com/yetone/magpie/blob/main/docs/reference.md> |
| 中文 README | <https://github.com/yetone/magpie/blob/main/README.zh-CN.md> |
| Discord 社区 | <https://discord.gg/vGSnD3ZKQF> |

---

## 目录

- [0. 10 分钟最短路径](#0-10-分钟最短路径)
- [1. magpie 是什么](#1-magpie-是什么)
- [2. 五个核心概念](#2-五个核心概念)
- [3. 安装前的准备](#3-安装前的准备)
- [4. 安装 magpie](#4-安装-magpie)
- [5. 第一步：添加 Provider（模型来源）](#5-第一步添加-provider模型来源)
- [6. 第二步：给每个 Agent 选模型](#6-第二步给每个-agent-选模型)
- [7. 第三步：验证真的跑通了](#7-第三步验证真的跑通了)
- [8. 方案（Profile）：一键切换整套配置](#8-方案profile一键切换整套配置)
- [9. 可选：路由组与自动故障转移](#9-可选路由组与自动故障转移)
- [10. 用量、花费与额度](#10-用量花费与额度)
- [11. 隐私与本地脱敏](#11-隐私与本地脱敏)
- [12. 命令行速查](#12-命令行速查)
- [13. 文件、日志与配置目录](#13-文件日志与配置目录)
- [14. 进阶功能（按需阅读）](#14-进阶功能按需阅读)
- [15. 常见问题 FAQ](#15-常见问题-faq)
- [16. 故障排查手册](#16-故障排查手册)
- [17. 卸载与恢复原样](#17-卸载与恢复原样)
- [18. 术语表](#18-术语表)
- [19. 官方链接汇总](#19-官方链接汇总)
- [附录 A：常用命令清单](#附录-a常用命令清单)
- [附录 B：三套最小可用配置](#附录-b三套最小可用配置)
- [附录 C：上线前检查清单](#附录-c上线前检查清单)

---

## 0. 10 分钟最短路径

如果你不想先读概念，照着做即可。这里给两条路线，选一条。

### 路线 A：我有一家厂商的 API Key（以 DeepSeek 为例）

```sh
# 1. 安装（macOS / Linux）
curl -fsSL https://usemagpie.ai/install.sh | sh

# 2. 添加 provider（把 sk-… 换成你的 key）
magpie provider add deepseek sk-…

# 3. 让 Claude Code 用 DeepSeek
magpie claude deepseek/deepseek-v4-pro

# 4. 验证
magpie ls
magpie provider test deepseek
```

然后**关掉旧的 Claude Code 会话，重新开一个终端**运行 `claude`。Agent 只在启动时读配置，旧会话不会变。

### 路线 B：我想用已登录的订阅（以 Claude 订阅为例）

1. 打开 magpie 应用。
2. 左侧点 **供应商（Providers）** → **＋ 添加供应商**。
3. 点 **Claude** 卡片，在浏览器里完成 Anthropic 登录。
4. 左侧点 **Agent** → 找到 **Claude Code** 那一行 → 点它当前的模型。
5. 在列表里选 `claude/…` 下的模型（例如 `claude/claude-opus-5-5`）。
6. 重开一个 Claude Code 会话。

### 最短验收

```sh
magpie ls          # 看每个 Agent 当前用的模型
magpie providers   # 看你添加的 provider
magpie usage today    # 用一次之后，看请求有没有经过网关
```

只要 `magpie usage` 里出现了刚才那次对话的 token，就说明请求确实走了 magpie。

> **注意**：如果你只用一个 Agent、只用它自家的订阅而且够用，magpie 对你不是必需的。它的价值在"多 Agent / 多 provider / 多账号"。

---

## 1. magpie 是什么

### 1.1 它解决的问题

没有 magpie 时，每个 Agent 都把模型写在自己的配置文件里，格式各不相同：

| Agent | 配置文件 | 格式 |
| :-- | :-- | :-- |
| Claude Code | `~/.claude/settings.json` | JSON，靠环境变量 |
| Codex | `~/.codex/config.toml` | TOML |
| Gemini CLI | `~/.gemini/settings.json` + `.env` | JSON + env |
| OpenCode | `~/.config/opencode/opencode.json(c)` | JSON |
| Crush | `~/.config/crush/crush.json` | JSON |

于是你面对这些麻烦：

- 想在 Claude Code 里用 Kimi，得手动改 `ANTHROPIC_BASE_URL` 和 token；
- 想在 Codex 里用 DeepSeek，得手动写 `[model_providers.*]`；
- 买了 ChatGPT 订阅，却只能在 Codex 里用；
- 一个账号额度用完了，要手动去另一个账号切换；
- 想知道这个月各个 Agent 花了多少钱，只能分别去各家后台看。

magpie 把这些收进**一个应用 + 一个本地网关**：

- **任意 Agent 用任意模型**，在一张表里选。
- **一个 provider 处处能用**：API Key、本地模型、你登录着的 Claude / ChatGPT / Copilot / Gemini / Grok 订阅。
- **额度用完自动接上**：同一个服务商的下一个账号、路由组里的下一个模型。
- **密钥不出你的电脑**：请求直接发给你选的供应商，不经过 magpie 的服务器。
- **统一用量**：token、缓存命中、花费、额度，按 Agent 和模型统计。

### 1.2 它是怎么工作的

magpie 启动时会在本机开一个网关，默认地址：

```
http://127.0.0.1:3425
```

你的 Agent 本来要发给自己厂商的请求，被 magpie 写进配置后改为发到这个本地网关。网关再把请求转给你选的 provider——中间会做协议转换、路由、重试、统计。

```
┌────────────────┐
│  Claude Code   │──┐
├────────────────┤  │
│  Codex         │──┤    HTTP / SSE
├────────────────┤  ├──────────────►  127.0.0.1:3425  ──────────────►  你的供应商
│  Gemini CLI    │──┤                   magpie 网关                        ├─ 订阅账号
├────────────────┤  │                   · 协议转换                          ├─ API Key 厂商
│  OpenCode      │──┤                   · 路由 / 故障转移                   ├─ 本地模型
├────────────────┤  │                   · 用量统计                          └─ 自定义地址
│  其他 Agent    │──┘                   · 可选脱敏
└────────────────┘
```

网关同时支持四种 API，并在它们之间互相转换：

| API | 典型 Agent |
| :-- | :-- |
| OpenAI Chat Completions | 大多数 OpenAI 兼容工具 |
| OpenAI Responses | Codex |
| Anthropic Messages | Claude Code |
| Google Gemini | Gemini CLI |

流式输出、工具调用、推理（reasoning）都支持。所以 **Agent 用哪种 API 不重要**，magpie 会把它转成目标模型能听懂的格式。

### 1.3 支持哪些 Agent

magpie 只显示**你电脑上已经安装或配置过**的 Agent。常见的有：

Claude Code、Claude Desktop、Codex、Gemini CLI、Antigravity CLI、OpenCode、OpenChamber、MiMo Code、DevEco Code、Pi、oh-my-pi、StepCode、MothX、Goose、Cursor CLI、Cursor Private Inference、Zed、VS Code Chat、VS Code Insiders、VSCodium Chat、JetBrains Air、Copilot（JetBrains）、Copilot CLI、Crush、DeepSeek Harness、Command Code、Kimi Code、Qwen Code、MiniMax Code、Droid、Cline、Qoder、Qoder CN、Grok Build、Grok App、ZCode、WorkBuddy、CodeBuddy、Raycast、OpenClaw、AstrBot……

完整列表见官方 README。对于"不在列表里、但能填 base URL"的工具，也可以直接用网关地址接入（见 [7.4](#74-非-magpie-管理的工具怎么接)）。

### 1.4 支持哪些平台

- **macOS**：菜单栏应用 + 窗口，已签名并公证，自动更新。
- **Windows**：窗口 + 托盘，暂未签名（首次运行 SmartScreen 可能提示）。
- **Linux**：桌面版（GTK 3 + WebKitGTK 4.1）或终端版（TUI + Web + CLI）。
- **Docker**：`ghcr.io/yetone/magpie`，适合服务器 / NAS。
- **Termux**：Android 终端版。

### 1.5 它不是什么

- **不是模型提供商**：它不卖 token，也不转发到自己的服务器。你的请求直接去你配的供应商。
- **不是 Agent**：它不会替你写代码，只负责"把模型接到 Agent 上"。
- **不是必须常驻的云服务**：但通过 magpie 用的模型需要网关在运行；关掉窗口后它还在菜单栏/托盘。
- **不会改你的其他配置**：只改它需要的那几个键，注释、顺序、缩进都保留，写入是原子的。

---

## 2. 五个核心概念

先记住这五个词，后面所有操作都围绕它们。

### 2.1 Provider（供应商）

**模型从哪里来。** 四类：

| 类型 | 例子 | 需要什么 |
| :-- | :-- | :-- |
| 订阅登录 | Claude Pro/Max、ChatGPT Plus/Pro、Copilot、Gemini Code Assist、Antigravity | 在 magpie 里点卡片登录 |
| 厂商预设 + API Key | Anthropic、OpenAI、DeepSeek、Kimi、智谱 GLM、MiniMax、Qwen、Mistral、Groq、xAI、OpenRouter…… | 粘贴 key |
| 本地模型 | Ollama、LM Studio | 本机服务在跑，无需 key |
| 自定义兼容地址 | 中转站、自建网关、另一台电脑上的 magpie | OpenAI 兼容 `/v1` 或 Anthropic 兼容根地址 |

每个模型都写作：

```
provider/model
```

例如：

```
deepseek/deepseek-v4-pro
moonshot/kimi-k2.5
claude/claude-opus-5-5
codex/gpt-5.5
```

### 2.2 Agent（智能体）

**使用模型的工具。** Claude Code、Codex、Gemini CLI、OpenCode 等，每个在 **Agent** 页占一行，显示它当前用的模型。

magpie 对每个 Agent 做两件事：

1. 在它的配置里写入一个指向本地网关的 provider；
2. 把它的模型设置改成 `provider/model`。

改完后，Agent 以为自己在调用原来的厂商，实际上请求发到了 `127.0.0.1:3425`。

### 2.3 Gateway（网关）

**本地地址 `127.0.0.1:3425`。** 它做四件事：

- 接收 Agent 的请求；
- 按模型名找到对应 provider；
- 在需要时转换 API 格式；
- 转发、重试、记录用量。

网关默认只监听本机回环地址，所以其他电脑访问不到。要让局域网里的其他电脑用，需要打开"局域网共享"并给每台电脑发网关密钥（见 [14.3](#143-远程-magpie另一台电脑用同一套模型)）。

### 2.4 Routing group（路由组）

**把多个模型/账号合成一个，给 Agent 当做一个模型来选。** 名字写作：

```
group/<id>
```

例如 `group/daily-coding`。当第一个模型额度用完、被限流或出错时，自动换下一个。

路由组是**可选的**。只有一个 provider、一个账号时，你不需要它。

### 2.5 Profile（方案）

**所有 Agent 配置的一份命名快照。** 例如：

- `省钱`：全部 Agent 用便宜模型；
- `专注`：Claude Code 用 Opus，Codex 用 GPT；
- `公司网络`：全部走公司允许的 provider。

点一下就能整套切换。

### 2.6 概念关系图

```
Provider ──提供──► Model（provider/model）──被选用──► Agent
   │                    │
   │                    └──可组合──► Routing group（group/<id>）──被选用──► Agent
   │
   └──账号/密钥可多个──► 自动故障转移

Profile = 所有 Agent 当前模型选择的一份快照
Gateway = 所有请求的必经之路（127.0.0.1:3425）
```

---

## 3. 安装前的准备

### 3.1 确认你的 Agent 已经装好

桌面版 magpie 不会替你安装 Agent，它只配置**已经存在**的 Agent（Docker 里的 Claude 订阅是例外：需要时会按需下载 Claude Code）。先确认你要用的命令能跑：

```sh
claude --version
codex --version
gemini --version
opencode --version
```

装好并至少登录过一次，magpie 才能发现它。

### 3.2 备份现有配置（强烈建议）

magpie 的写入是原子的、只动它需要的键，但第一次上手仍建议先备份：

```sh
# macOS / Linux
cp ~/.claude/settings.json ~/.claude/settings.json.bak 2>/dev/null
cp ~/.codex/config.toml ~/.codex/config.toml.bak 2>/dev/null
cp ~/.config/opencode/opencode.json ~/.config/opencode/opencode.json.bak 2>/dev/null
```

Windows 对应的文件在：

```
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\config.toml
%APPDATA%\opencode\opencode.json
```

> 即使不备份，也可以用 `magpie <agent> default` 让 magpie 删除自己写入的内容并恢复被替换的值（例如你原来的 `ANTHROPIC_BASE_URL`）。

### 3.3 检查网络与代理

- **安装 / 更新**要从 GitHub 下载发布文件。连不上时用 `--proxy` 或 `--mirror`（见 [4.4](#44-安装脚本参数)）。
- **调用云端模型**需要能访问对应厂商。
- **本地模型**（Ollama / LM Studio）不需要外网。
- **代理软件**（Surge、Clash、mihomo 等）如果开了系统代理、TUN 或增强模式，可能把发往 `127.0.0.1:3425` 的请求也接走，导致 Agent 报 502 而 magpie 的路由页什么都没有。解决办法见 [16.2](#162-agent-报-502但路由页什么都没有)。

### 3.4 磁盘与权限

- 应用本身不大；如果你要启用**本地脱敏模型**，会额外下载约 **39 MB**（AstrLink Guard，默认）或约 **1.6 GB**（OpenAI Privacy Filter，可选）。
- Docker 方式需要一个数据卷挂载到 `/config`。
- 配置目录 `~/.config/magpie` 里的 `providers.json`、`logins.json` 等是 `0600`（只有你自己可读）。
- Linux 安装脚本在缺 WebKitGTK 4.1 时可能用 `sudo` 调包管理器安装依赖，会提示你输入密码。

### 3.5 先决定：你要用哪种方式

| 你的情况 | 推荐方式 |
| :-- | :-- |
| 日常在 Mac/Windows 桌面用 | 桌面应用 |
| 只在终端里工作 | `magpie tui` 或 `magpie web` |
| 服务器 / NAS，给多台电脑提供模型 | Docker（见 [14.4](#144-docker-与-nas)） |
| 想从源码构建 / 参与开发 | `go install` 或 `make build`（见 [4.6](#46-从源码安装)） |

---

## 4. 安装 magpie

### 4.1 macOS

1. 打开 <https://usemagpie.ai/zh/>，下载 macOS 版。
2. 按官网提示安装（Mac 版已签名并公证）。
3. 首次启动后，菜单栏会出现 magpie 图标，同时打开主窗口。
4. 关掉窗口不会退出，它仍在菜单栏运行；需要完全退出时用菜单里的退出。

也可以用安装脚本：

```sh
curl -fsSL https://usemagpie.ai/install.sh | sh
```

macOS 版会自动更新：应用在后台下载新版本，重启或退出时安装；菜单里有 **Restart to Update**。

### 4.2 Windows

1. 从官网下载 Windows 版并安装。
2. 目前 Windows 构建**尚未签名**，首次运行 SmartScreen 可能提示，选择"更多信息 → 仍要运行"。
3. 安装后出现在托盘；可以在设置里开启登录时自动启动（`magpie autostart on`）。
4. Windows 上的 magpie 也能配置 **WSL 发行版里的 Agent**，会话和用量一样统计。

### 4.3 Linux

一条命令：

```sh
curl -fsSL https://usemagpie.ai/install.sh | sh
```

脚本会自动判断装哪种版本：

| 情况 | 装什么 |
| :-- | :-- |
| 有桌面，且已装 GTK 3 + WebKitGTK 4.1 | 桌面版（窗口、托盘、应用菜单） |
| 有桌面，但缺 WebKitGTK 4.1 | 先用系统包管理器（apt / dnf / pacman / zypper）通过 `sudo` 装依赖，再装桌面版 |
| 没有桌面、没有终端、没有 sudo、发行版不认识 | 终端版（`magpie tui`、`magpie web`、CLI），并告诉你该装哪个包 |
| Alpine（musl） | 总是终端版 |

想自己选：

```sh
# 强制桌面版（即使库现在装不上）
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --gui

# 强制终端版，不装任何系统包
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --tui
```

`MAGPIE_CLI=1` 等价于 `--tui`。

Debian / Ubuntu 上缺库时通常是：

```sh
sudo apt install libwebkit2gtk-4.1-0
```

Fedora：

```sh
sudo dnf install webkit2gtk4.1
```

装好库后再跑一次安装脚本，就会换成桌面版。

**Termux（Android）：** 安装脚本会把 Android 终端版放进 `$PREFIX/bin`；用 `magpie web` 或 `magpie tui` 打开界面；`magpie autostart on` 需要安装 Termux:Boot。

### 4.4 安装脚本参数

| 参数 / 环境变量 | 作用 |
| :-- | :-- |
| `--gui` | 强制安装桌面版 |
| `--tui` | 强制安装终端版，不装系统包 |
| `--proxy <url>` | 安装时走代理（http、https、socks5、socks5h） |
| `--mirror <前缀>` | 从你指定的 GitHub 下载镜像取发布文件 |
| `--uninstall` | 卸载脚本安装的内容，保留 `~/.config/magpie` 里的设置 |
| `--purge` | 连同设置、provider、用量一起删除，并断开已连接的 Agent |
| `MAGPIE_CLI=1` | 同 `--tui` |
| `MAGPIE_PROXY` | 同 `--proxy` |
| `MAGPIE_MIRROR` | 同 `--mirror` |
| `HTTPS_PROXY` / `ALL_PROXY` | 标准代理环境变量，同样生效 |

示例：

```sh
# 墙内 / 公司网络
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --proxy http://127.0.0.1:7890

# 用镜像
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --mirror https://mirror.example/

# 卸载但保留设置
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --uninstall

# 彻底删除
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --uninstall --purge
```

> 用镜像时，每个文件的 SHA-256 仍然来自 usemagpie.ai，不是镜像；被镜像改过的文件会被拒绝安装。

### 4.5 Docker（先看一眼，细节在第 14 章）

```sh
docker run -d --name magpie --restart unless-stopped \
  -p 127.0.0.1:3425:3425 -p 127.0.0.1:3430:3430 \
  -e MAGPIE_WEB_KEY=$(openssl rand -hex 16) \
  -v magpie-config:/config \
  ghcr.io/yetone/magpie:latest web --addr 0.0.0.0:3430 --no-open
```

- `3425`：Agent 调用的网关。
- `3430`：浏览器管理界面。
- `/config`：所有数据（provider、key、登录、插件、用量）。
- 容器外的请求必须带**网关密钥**，否则 401。

完整步骤、Compose、NAS 配置见 [14.4](#144-docker-与-nas)。

### 4.6 从源码安装

```sh
go install github.com/yetone/magpie@latest
```

或者：

```sh
git clone https://github.com/yetone/magpie.git
cd magpie

make build      # 当前平台的桌面版（需要 cgo + 平台 webview）
make app        # macOS：magpie.app，菜单栏应用，无 Dock 图标
make cli        # 纯终端版，不需要 cgo，可交叉编译
make release    # dist/：本机原生应用 + 各平台 CLI
```

Linux 构建桌面版需要 `libgtk-3-dev` 和 `libwebkit2gtk-4.1-dev`；Termux 用 `-tags nogui`，然后用 `magpie web` 或 `magpie tui`。

### 4.7 第一次启动

| 方式 | 命令 | 说明 |
| :-- | :-- | :-- |
| 桌面应用 | 打开 magpie | 窗口 + 菜单栏图标 |
| 只要菜单栏 | `magpie tray` | 适合放进登录项 |
| 终端界面 | `magpie tui` | 全功能，键盘操作 |
| 浏览器界面 | `magpie web` | 适合 WSL / SSH 服务器 |
| 只要网关 | `magpie serve` | 无窗口，供 Agent 调用 |
| 后台服务 | `magpie daemon install` | systemd / launchd，登录时自启；Linux 上无人登录也要运行需 `sudo loginctl enable-linger $USER` |

第一次启动后，应用会：

1. 扫描本机已安装的 Agent；
2. 在 `127.0.0.1:3425` 启动网关；
3. 显示主窗口。

### 4.8 界面导览

主窗口通常有这些标签页（不同版本可能多出"会话""资源库"等）：

| 标签 | 作用 |
| :-- | :-- |
| **供应商 / Providers** | 添加、编辑、测试 provider；看谁在用哪些模型 |
| **Agent** | 每个 Agent 一行，点模型即可切换 |
| **路由 / Routing** | 路由组、规则、实时决策 |
| **用量 / Usage** | token、花费、缓存、额度、上下文 |
| **网关 / Gateway** | 网关地址、网关密钥、接入示例 |
| **插件 / Plugins** | 登录插件、网关中间件 |
| **设置 / Settings** | 隐私、网络与共享、同步、可观测性等 |

菜单栏面板（托盘）可以快速看：

- 当前各 Agent 用的模型；
- 各订阅剩余额度；
- 当前方案（Profile）；
- 快速切换模型。

**TUI 键盘操作：**

| 键 | 作用 |
| :-- | :-- |
| `↑` `↓` | 选择 Agent |
| `←` `→` | 选择字段（模型、推理强度等） |
| `↵` | 打开选择器 |
| 输入 | 筛选；回车接受自定义值 |
| `s` | 把当前配置保存为方案 |
| `p` | 应用 / 删除方案（`ctrl+d` 删除） |
| `S` | 同步模型目录 |
| `q` | 退出 |

---

## 5. 第一步：添加 Provider（模型来源）

打开 **供应商** 标签页，点 **＋ 添加供应商**。你会看到四类入口：订阅、厂商预设、本机、自定义 URL。

### 5.1 订阅登录（无需 API Key）

magpie 支持直接登录你已有的订阅，把它当成 provider 给所有 Agent 用。

**支持直接点卡片登录的：**

- Claude（Pro、Max、Team）
- ChatGPT（Plus、Pro、Business）
- GitHub Copilot
- Gemini（Code Assist）
- Antigravity

**通过插件页添加的：**

- Cursor
- Grok（SuperGrok）
- Devin
- 以及任何 OpenCode 登录插件 / pi 包

**操作步骤：**

1. 供应商 → ＋ 添加供应商；
2. 点对应卡片；
3. 浏览器会打开厂商自己的登录页；
4. 完成登录后，账号立刻出现在 magpie 里。

Copilot 会给你一个验证码，需要在 GitHub 页面上输入。

**几个要点：**

- **多个账号**：再点一次同一张卡片，就能再加一个账号；它们都列在这个 provider 下面，点一下切换用哪个。
- **已经登录过**：本机上已经登录的 Agent 也会作为 provider 出现，显示为 **已登录 …**。
- **模型前缀**：订阅里的模型在其他 Agent 的选择列表里写作：

```
claude/…
codex/…
copilot/…
antigravity/…
```

例如想在 Codex 或 Claude Code 里用 Antigravity 的 Claude / Gemini 模型：先在 magpie 里登录 Antigravity，再在 Agent 那一行选 `antigravity/…` 模型即可。

> **两个重要提醒（官方明确写出）：**
>
> 1. **Antigravity**：Google 可能封禁在 Antigravity 之外使用的 Antigravity 账号，请用"丢了也不心疼"的账号。
> 2. **Claude 订阅**：请求通过本机安装的 Claude Code 运行，所以请保留 Claude Code，不要卸载。

**订阅是怎么被发现的？**

magpie 会读取智能体的登录信息（Claude Code 和 Codex 的文件、钥匙串）来找到它们登录的订阅。不想让它读取，可以：

- 设置 → 隐私 → 关闭"订阅账号"；或
- `magpie accounts discovery off`（`on` 重新开启）；或
- 本次运行临时关闭：`MAGPIE_NO_ACCOUNTS=1`。

API Key 类型的 provider 不受影响。

### 5.2 厂商预设 + API Key

适合你已经有某家厂商 API Key 的情况。

**支持的预设（部分）：**

Anthropic、OpenAI、Google Gemini、Amazon Bedrock（runtime 与 Mantle）、DeepSeek、Kimi、智谱 GLM、MiniMax、阶跃星辰、LongCat、通义千问、百度千帆、腾讯云、华为云 MaaS、火山方舟、Mistral、Groq、Nous Portal、xAI、OpenRouter、Together、Fireworks、硅基流动、NVIDIA NIM、魔搭、Ollama、LM Studio……

**操作步骤：**

1. 供应商 → ＋ 添加供应商；
2. 选择厂商预设；
3. 点"获取密钥 ↗"会打开厂商的密钥页面；
4. 粘贴 key，保存；
5. magpie 会向厂商询问它提供哪些模型，并列出来；
6. 展开 provider 那一行，可以选择给 Agent 暴露哪些模型；
7. 点 **测试** 发一个最小请求，确认 key 可用。

**命令行等价操作：**

```sh
magpie presets                       # 查看内置预设
magpie provider add deepseek sk-…    # 预设只需要 key
magpie providers                     # 查看你的 provider
magpie provider test deepseek        # 测试
magpie provider models deepseek      # 重新拉取模型列表
```

> **Key 存在哪？** `~/.config/magpie/providers.json`，权限 `0600`，只有你自己可读。magpie **从不读取 shell 环境变量里的 key**：你添加什么，它就用什么。

### 5.3 本地模型（Ollama / LM Studio）

无需 API Key，也不需要联网。

1. 先在本地把 Ollama 或 LM Studio 跑起来，并拉取至少一个模型；
2. 供应商 → ＋ 添加供应商 → 选 **本机** 下的 Ollama 或 LM Studio；
3. magpie 会读取本地服务提供的模型列表；
4. 在 Agent 那一行选 `ollama/…` 或 `lmstudio/…` 模型。

**命令行：**

```sh
magpie provider add ollama
# LM Studio 在界面里选择；CLI 的 provider id 以 magpie presets 的输出为准
```

> 本地模型默认走本机回环地址，不需要网关密钥；如果 provider 请求失败，先确认本地服务端口和模型名。

### 5.4 自定义兼容地址（中转站 / 自建网关 / 远程 magpie）

其他接口都走这里。你需要准备：

- **名称**：随便起，会变成模型前缀的一部分；
- **OpenAI 兼容地址**：以 `/v1` 结尾；
- **Anthropic 兼容地址**：根地址，即 `ANTHROPIC_BASE_URL` 的值（不带 `/v1`）；
- 两者可以只填一个，也可以都填；
- **密钥**；
- 可选：要暴露的模型 ID 列表。

**操作步骤：**

1. 供应商 → ＋ 添加供应商 → 选择 **自定义**；
2. 填名称、地址、密钥；
3. 如果服务商支持多种 API，就都填上；每个 Agent 用它原生的那种，其余由 magpie 转换；
4. 保存后 magpie 会拉取模型列表；也可以手动填 `models=`。

**命令行：**

```sh
magpie provider add "My Relay" \
  url=https://relay.example.com/v1 \
  key=sk-… \
  models=gpt-5.5,claude-sonnet-5
```

> **地址安全要求（官方）：** Base URL 必须是 https；只有本机或局域网地址才允许明文 http。

**给供应商的怪名字起别名：**

有些中转站用自己的 id 提供模型，比如 `deepseek-v4.1-flash-1017`、`grok-4.6@500k`。你可以给它起一个干净的 id：

```sh
magpie model id relay/grok-4.6@500k grok-4.6
magpie model ids
magpie model id relay/grok-4.6 --reset
```

之后 magpie 列出、路由、合并 `auto/grok-4.6` 组、限制网关密钥、记录用量，都用 `grok-4.6`；向供应商请求时仍然发 `grok-4.6@500k`。

### 5.5 导入已有配置

如果你之前在 Claude Code、Codex、CC Switch、Alma 里配过 provider，不用重新填。

**应用内：**

- **供应商** 页顶部的 **导入…** 会读取：
  - Claude Code 的 `settings.json`
  - Codex 的 `config.toml`
  - OpenCode 的 `auth.json`（打开导入对话框时读取）
  - CC Switch、Alma 等本地配置
- 它**不会修改**这些文件，只是复制进来；
- 勾选你要的，确认后才添加。

**注意：**

- 导入的 provider 是一份**拷贝**；之后在 Agent 里的改动不会再同步过来。
- 指向 magpie 自己的条目、只写 `env_key` 的条目会被跳过。
- OpenCode 的登录（Claude Pro/Max、Copilot 等）会被列出但跳过——订阅请在 magpie 里重新登录。

**厂商的一键导入链接：** 有些厂商或中转站会给你一个 `magpie://import?...` 链接。打开后 magpie 会显示将要添加的名称、地址、模型和 key 去向，确认后才保存；`magpie import <link>` 在终端做同样的事。

### 5.6 一个 provider 下放多个账号 / 密钥

展开 provider 那一行，勾选要用的每个账号或密钥。然后设置：

- **路由**：请求怎么分给它们（智能、按顺序、用完再换、轮流、用量少优先、到期前用完、按权重、速度优先）；
- **备用**：这个 provider 额度用完、被限流或不可用时，改用哪些模型。

只有一个账号时，可以跳过。

### 5.7 验证 provider 是否可用

```sh
magpie providers
magpie models
magpie provider test deepseek
```

或者在应用里：

- 供应商页 → 展开 provider → **测试**；
- 成功会显示延迟；
- 失败时先检查 key、base URL、余额、代理。

### 5.8 关于密钥安全

- Key 保存在本机 `providers.json`（`0600`）。
- 请求直接发给你选的供应商，不经过 magpie 的服务器。
- 备份时默认包含 key；不想备份 key 用 `magpie backup --no-keys`。
- 用第三方中转站时，你的 prompt 和 key 都会经过中转站——只选你信任的。
- 可以在 **设置 → 隐私** 打开本地脱敏，让 key、密码、邮箱在离开电脑前被替换（见第 11 章）。

---

## 6. 第二步：给每个 Agent 选模型

### 6.1 操作步骤

1. 左侧点 **Agent**；
2. 找到你要改的 Agent（例如 **Claude Code**）；
3. 点它当前模型那一格；
4. 弹出可搜索的模型列表；
5. 输入关键词筛选，或直接输入列表里没有的模型 ID；
6. 选中后，magpie 立刻改写该 Agent 的配置文件。

**列表分组：**

- Agent 自带的模型；
- 你的路由组；
- 你添加的每个 provider 的模型。

### 6.2 模型命名规则

| 形式 | 含义 | 例子 |
| :-- | :-- | :-- |
| `provider/model` | 某个 provider 的某个模型 | `deepseek/deepseek-v4-pro` |
| `group/<id>` | 路由组 | `group/daily-coding` |
| `auto/<model>` | 自动合并的同名模型组 | `auto/claude-opus-4.6` |
| `office/…` | 远程 magpie 提供的模型 | `office/deepseek/deepseek-v4-pro` |

在 Agent 的选择器里，订阅模型的写法是：

```
claude/…
codex/…
copilot/…
antigravity/…
```

### 6.3 推理强度与快速模式

**推理强度（effort）：**

- Agent 有这个选项时（Codex 的"推理强度"、Pi 的"思考"），列表下方会出现滑块；
- 选好后会写进该 Agent 的配置。

**快速模式（Fast）：**

- 有快速模式的模型，在列表里那一行的星标旁有一个闪电按钮；
- 包括：ChatGPT（Codex）账号的 GPT 模型、OpenAI Key 的 GPT 和 o 系列、有快速版本的 Cursor 模型、Anthropic Key 的 Claude Opus，以及插件或另一个 magpie 说明支持 priority 的模型；
- 点一下，这个 Agent 发给这个模型的每个请求都会用服务商的快速模式；
- **更快，但价格更高**；
- 不管 Agent 用哪种 API，都会加上 `service_tier: "priority"`；Claude 则是 `speed: "fast"`；
- ChatGPT 账号提供 Ultrafast 时，再点一下是 Ultrafast，再点一下恢复普通速度；
- 每个 Agent 分开设置；
- 路由组里每个成员那一行也有同样的开关。

**Claude Code 的分档模型：**

Claude Code 通过 magpie 使用模型时，`opus`、`sonnet`、`haiku` 各档还可以单独指定模型，默认跟随主模型。

```sh
magpie claude haiku deepseek/deepseek-v4-flash   # 只把 haiku 档换成 DeepSeek
magpie claude haiku ""                            # 恢复跟随主模型
```

### 6.4 magpie 改了哪些文件

这是新手最关心的部分。magpie **只改它需要的键**，保留注释、顺序、缩进，写入是原子的。

| Agent | 写入位置 | 写入内容 |
| :-- | :-- | :-- |
| Claude Code | `~/.claude/settings.json` | `env` 里的 `ANTHROPIC_BASE_URL`、`ANTHROPIC_AUTH_TOKEN` 和模型变量 |
| Codex | `~/.codex/config.toml` | 一个 `[model_providers.magpie]` 表和模型；模型列表写在 `magpie-models.json`。你的 ChatGPT 登录不受影响 |
| OpenCode、MiMo Code、DevEco Code、Pi、Crush | 各自配置文件 | 一个 `magpie` provider 条目，以及 `magpie/provider/model` |
| Gemini CLI | `~/.gemini/settings.json` + `.env` | `GOOGLE_GEMINI_BASE_URL` 指向网关、API 密钥认证、模型 |
| AtomCode | `~/.atomcode/config.toml` | 一个 `magpie` provider 账号，以及每个模型各自的条目 |
| Snow CLI | `~/.snow/profiles/magpie.json` | 一个 `magpie` 配置文件，并设为当前配置 |
| QwenPaw | `~/.qwenpaw.secret/providers/custom/magpie.json` + 每个 agent 的 `agent.json` | `magpie` provider + `active_model`；首次连接后需重启 QwenPaw |
| Claude Desktop | `~/Library/Application Support/Claude*` 等 | 第三方网关模式；需重启 Desktop |
| VS Code Chat | `Code/User/settings.json` + `chatLanguageModels.json` | `magpie` Custom Endpoint 组；VS Code 1.122+ |
| Cursor CLI | `~/.cursor/cli-config.json` | 模型 |
| Copilot CLI | `~/.copilot/settings.json` | 模型 |
| Zed | `~/.config/zed/settings.json` | 一个 `magpie` OpenAI 兼容 provider |

> 更完整的 Agent / 文件 / 字段对照见官方英文参考文档的 **Agents** 表。

### 6.5 什么时候生效

**Agent 在启动时读取配置。** 所以：

- 已经在运行的会话，保持它启动时的模型；
- 改完要**开一个新会话**；
- **Codex** 还要重启 Codex 应用和已经打开的 `codex` 会话（它在启动时生成模型列表）；
- Claude Desktop 需要重启；
- Paseo、编辑器插件等前端启动的是同一个 CLI，读的是同一份配置，所以从下一个会话起也会用上新模型。

### 6.6 用命令行设置

```sh
magpie ls                                  # 列出每个 Agent 及当前设置
magpie claude deepseek/deepseek-v4-pro     # Claude Code 用 DeepSeek
magpie codex moonshot/kimi-k2.5            # Codex 用 Kimi
magpie opencode anthropic/claude-sonnet-5  # OpenCode 用 Claude
magpie codex effort high                   # 设置推理强度
magpie codex default                       # 恢复 Codex 自己的默认值
```

Agent 名字支持前缀：

```sh
magpie cc deepseek/deepseek-v4-pro    # cc = Claude Code
magpie oc anthropic/claude-sonnet-5   # oc = OpenCode
magpie gem …                          # gem = Gemini CLI
```

### 6.7 恢复某个 Agent 的原样

两种方式：

1. 在模型选择器里选 Agent **自带的模型**；
2. 运行：

```sh
magpie claude default
magpie codex default
```

magpie 会删掉它写入的内容，并恢复被它替换掉的值（例如你原来的 `ANTHROPIC_BASE_URL`）。

### 6.8 几个 Agent 的特别说明

**Claude Code**

- 通过 `~/.claude/settings.json` 的 `env` 接入；
- 使用 Claude 订阅时，请求通过本机 Claude Code 运行，所以不要卸载 Claude Code；
- 可以单独设置 opus / sonnet / haiku / fable 档。

**Codex**

- 通过 `~/.codex/config.toml` 的 `[model_providers.magpie]` 接入；
- 你的 ChatGPT 登录不受影响；
- 切换模型后要重启 Codex 应用。

**Gemini CLI**

- 通过 `GOOGLE_GEMINI_BASE_URL` 指向网关；
- 使用 API Key 认证。

**OpenCode / MiMo Code / DevEco Code / Pi / Crush**

- 写入一个名为 `magpie` 的 provider；
- 模型写作 `magpie/provider/model`。

**VS Code / Cursor / Zed 等**

- magpie 会写一个自定义 endpoint / provider；
- 具体入口在各自设置里，重启编辑器或重新加载窗口后生效。

---

## 7. 第三步：验证真的跑通了

不要只看"配置写进去了"，要确认请求真的经过网关并拿到回复。

### 7.1 三条命令

```sh
# 1. 看 Agent 当前设置
magpie ls

# 2. 看 provider 是否可用
magpie provider test deepseek

# 3. 用一次之后看用量
magpie usage today
```

`magpie ls` 应该能看到目标 Agent 和它的 `provider/model`。输出类似下面这样（**示意，实际格式以你的版本为准**）：

```
Claude Code   deepseek/deepseek-v4-pro   …
Codex         moonshot/kimi-k2.5         …
```

### 7.2 在 Agent 里验证

1. 关掉旧会话；
2. 新开一个终端；
3. 运行 Agent（例如 `claude`）；
4. 随便问一句；
5. 回到 magpie 的 **用量** 页，或运行 `magpie usage today`，确认 token 数增加。

> 不要只问"你是什么模型"来判断——很多模型会答错自己的名字。**看用量页/网关请求记录**才可靠。

### 7.3 看网关与日志

- **网关页**：最近调用、请求状态、延迟；
- **路由页**：每个请求由哪个 provider / 账号回答、为什么；
- **日志**：

```sh
magpie log          # 日志路径
magpie log -n 200   # 最后 200 行
magpie log -f       # 持续跟随
```

日志会记录 provider 拒绝请求、重试、插件启动失败等；**不会**记录你的密钥、提示词或回复。

### 7.4 非 magpie 管理的工具怎么接

任何能填 base URL 的工具都可以直接用网关：

```sh
export OPENAI_BASE_URL=http://127.0.0.1:3425/v1
export OPENAI_API_KEY=magpie

export ANTHROPIC_BASE_URL=http://127.0.0.1:3425
export ANTHROPIC_API_KEY=magpie

export GOOGLE_GEMINI_BASE_URL=http://127.0.0.1:3425
export GEMINI_API_KEY=magpie
```

> 这些环境变量**只用于 magpie 不管理的工具**。对 magpie 列表里的 Agent，不需要手动 export——magpie 会把地址和令牌直接写进它们的配置文件。

**网关提供的路由：**

| 路径 | API |
| :-- | :-- |
| `/v1/chat/completions` | OpenAI Chat Completions |
| `/v1/responses` | OpenAI Responses（HTTP / SSE / WebSocket） |
| `/v1/messages` | Anthropic Messages |
| `/v1/messages/count_tokens` | Anthropic token 计数 |
| `/v1beta/models/{model}:generateContent` | Google Gemini |
| `/v1/models`、`/v1beta/models` | 模型目录 |

### 7.5 验收清单

| 检查项 | 期望结果 |
| :-- | :-- |
| `magpie ls` | 目标 Agent 显示你选的 `provider/model` |
| `magpie models` | 列表里包含该模型 |
| `magpie provider test <id>` | 返回成功和延迟 |
| 新开 Agent 会话 | 能正常对话 |
| `magpie usage today` | 出现这次请求的 token |
| 网关页 | 能看到这次请求 |
| 日志 | 没有持续报错 |

全部通过，第一步配置就完成了。

---

## 8. 方案（Profile）：一键切换整套配置

### 8.1 它是什么

方案是**所有 Agent 配置的一份命名快照**。比如：

- `省钱`：Claude Code 用 DeepSeek，Codex 用便宜的模型；
- `专注`：Claude Code 用 Opus，Codex 用 GPT；
- `公司`：全部走公司允许的 provider。

每个 Agent 还可以有自己的常用模型短名单，方案也会一并保存。

### 8.2 在应用里使用

1. 把所有 Agent 的模型调好；
2. 到 **Agent** 页底部，点 **＋ 保存当前**；
3. 起个名字，例如 `work`；
4. 之后点这个方案的 chip，就能一键全部切回；
5. 正在使用的方案带 ✓；
6. 手动改过设置后，✓ 会消失。

菜单栏面板的 **方案** 按钮也会显示当前方案名。

### 8.3 用命令行

```sh
magpie save work      # 保存当前配置为 work
magpie use work       # 应用 work
magpie profiles       # 列出所有方案
magpie rm work        # 删除
```

### 8.4 适合的场景

- 白天用强模型、晚上用便宜模型；
- 公司网络只允许某些 provider；
- 不同项目用不同模型组合；
- 给同一台电脑的不同使用者准备不同配置。

> 方案保存的是**Agent 的模型选择**，不是 provider 和 key；换电脑时 provider 需要另行同步（见 14.5 备份与同步）。

---

## 9. 可选：路由组与自动故障转移

> **什么时候需要？** 只有一个 provider、一个账号时，可以跳过整章。等你有了多个账号、多个 key，或者同一个模型在多个 provider 上都有时，再回来看。

### 9.1 路由组是什么

路由组是**几个模型（来自一个或多个 provider）**，Agent 把它当作一个模型来选：

```
group/<id>
```

例如：

```
group/daily-coding
```

网关会把请求分摊到每个成员的账号和 key 上；一个失败时换下一个。

### 9.2 自动组：不用手动建

只要两个 provider 提供**同名模型**，magpie 就会自动把它们组成一个路由组：

```
auto/<模型 id>
```

例如：

- Claude 订阅和 Copilot 都有 Claude Opus → `auto/claude-opus-4.6`
- DeepSeek 官方 API 和 OpenRouter 都有 DeepSeek → `auto/deepseek-…`

名字按各家不同的写法匹配（`deepseek/deepseek-chat` 和 `deepseek-chat` 算同一个）。

给 Agent 选这个组，请求就会分摊到这些 provider 上。不想要的可以隐藏：

```sh
magpie group rm auto/claude-opus-4.6
```

### 9.3 手动建组

**应用内：**

1. 打开 **路由** 标签页；
2. 点 **新建组**；
3. 起名，添加模型，选路由方式；
4. 保存。

**命令行：**

```sh
magpie group add "Opus anywhere" \
  models=claude/claude-opus-5-5,copilot/claude-opus-5.5 \
  routing=smart

magpie claude group/opus-anywhere
```

### 9.4 路由模式

| 模式 | 谁排在前面 | 适合 |
| :-- | :-- | :-- |
| **`smart`**（默认） | 在还有余量的订阅里，额度最先重置的排第一；重置时浪费最少 | 大多数情况 |
| **`order`** | 先用第一个模型，它答不了再换下一个 | 主备关系明确 |
| **`drain`**（用完再换） | 一个账号/key 用到额度用完，再换下一个；前面的恢复了也不切回去 | 想把某个账号彻底用完 |
| **`rotate`** | 会话的每一轮交给下一个成员 | 均摊负载 |
| **`usage`** | 剩余额度最多的排第一 | 避免某个账号先耗尽 |
| **`pace`** | 每周剩余额度 ÷ 距重置的小时数，最高的排第一 | 让周额度在重置前用得最均匀 |
| **`weight`** | 按权重分配请求（1–1000），均匀穿插 | 主账号多用、备用少用 |
| **`fast`**（速度优先） | 最近回答最快的成员先用（首字时间 + 输出速度） | 多通道比速度 |
| **`cheap`**（最便宜优先） | 此刻每百万 token 价格最低的成员先用 | 成本敏感 |

> `cheap` 只适用于路由组；价格来自你设的价格或标价，再乘 provider 的价格倍率和分时倍率。

### 9.5 会话保持（stays）

决定一个对话在回答它的账号/key 上留多久：

| 值 | 行为 |
| :-- | :-- |
| **`auto`**（默认） | 一轮之内总是保持；跨轮则在厂商缓存还值得保留时保持 |
| **`session`** | 整个会话，只要它还能回答 |
| **`turn`** | Agent 回传工具结果时不换；你再次发言时重新决定 |
| **`off`** | 每个请求都重新路由 |
| **`strict`** | 绝不换到别的账号/key，即使超过用量上限；错误直接交给 Agent |

> 只要厂商的提示词缓存还值得保留，对话就留在回答它的那个账号上。这是默认行为，不要轻易改成 `off`，否则缓存命中率会下降、花费会上升。

### 9.6 规则：把某类对话交给指定模型

路由组的 **规则** 可以把符合条件的一轮对话交给组里指定的模型。条件包括：

| 条件 | 说明 |
| :-- | :-- |
| `tokens ≥ N` | 对话够长 |
| 带图片 | 请求里包含图片 |
| 要求推理 | 开启，或不低于某个强度 |
| 来自某些 Agent | 例如只在 Claude Code 里生效 |
| 意图 | 用几个词描述用户想做什么，由一个小模型判断 |
| 正在压缩上下文 | Claude Code 的 `/compact`、自动压缩等 |
| 在某个文件夹（项目）里 | Agent 的工作目录匹配 |

**推理强度：** 规则还可以设定它的模型用什么推理强度（规则的 ⋯ 菜单 → **设定推理强度**，或 CLI `use-effort=low`）。不管 Agent 要求什么，该模型都按这个强度发送；magpie 不知道档位的模型、以及排在后面做备用的模型，按 Agent 的要求发送。

**规则的行为：**

- 每条规则的条件要**同时满足**；
- 每次你发新消息时从上往下检查；
- 第一条命中的规则把它的模型排在最前，其余成员排在后面做故障转移；
- 没有命中任何规则的一轮，照组本来的方式路由；
- 规则只在**一轮开始时**决定；Agent 回传工具结果的过程中不会半途换模型（除非这一轮涨到超过所在模型的上下文）。

**意图路由：**

- 每一轮开始时，magpie 会问一次组里指定的意图判断模型：用户这条消息属于哪个可能命中的意图；
- 建议选小而快、不带推理的模型；
- 只发送这一条消息，不包括整段对话；
- 出错、超过超时时间（默认 8 秒）或给不出答案时，意图条件不命中；
- 调用会计入用量。

**压缩上下文：**

- 命中后，让更便宜、更快的模型来写总结；
- 对话本身仍留在原来的模型上。

**时段和星期：**

- 规则可以只在某些时段、某几天生效，按这台电脑的时钟；
- 可以把某段时间的对话"先发给"别的模型；
- 也可以**暂停**某个模型：规则生效期间，这个模型移出分组——不排第一、不做备用、也不会因为会话粘性被留住；时段过了自动回来。

**文件夹：**

- `~/work/acme` 包括 `~/work/acme/web`，不包括 `~/work/acme-old`；
- 以 Agent 在请求里告诉模型的工作目录为准；
- 不说明工作目录的 Agent 不会命中任何按文件夹的规则；
- macOS 和 Windows 上不区分大小写；`C:\work` 也就是 WSL 里的 `/mnt/c/work`。

**命令行示例：**

```sh
# 长对话优先发给 OpenRouter
magpie group rule add opus-anywhere use=openrouter/anthropic/claude-opus-5.5 tokens=150k

# 简单问题交给便宜的模型
magpie group rule add opus-anywhere use=copilot/claude-opus-5.5 intent="一个简单问题" classifier=groq/llama-3.1-8b-instant

# /compact 的总结交给便宜的模型
magpie group rule add opus-anywhere use=deepseek/deepseek-v4-flash compact

# 这些项目里的对话先发给 Kimi
magpie group rule add opus-anywhere use=kimi/kimi-k2.6 folders=~/oss,"~/My Project"

# 这段时间不用 Copilot
magpie group rule add opus-anywhere pause=copilot/claude-opus-5.5 time=14:00-18:00 days=mon-fri

# 只留给某个文件夹
magpie group rule add opus-anywhere pause=claude/claude-opus-5-5 except-folders=~/big-task

# 查看规则
magpie group rule opus-anywhere
```

### 9.7 路由组里放路由组（嵌套）

- 成员也可以是另一个路由组；
- 在"添加模型"的 **路由组** 分类里选它，或用 `models+=group/<id>`；
- 子组占据它在顺序里的位置，在自己的模型之间按自己的路由方式和规则分配；
- **不会无限递归**：一个组不能包含它自己（直接或间接）；最多嵌套 **8 层**；
- 一个组被别的组包含时不能删除，要先从那个组里移除它；
- 每一层按自己的路由；外层不会伸进子组内部去混合排序；
- 子组里的模型全部失败时，由外层组的下一个成员接手。

### 9.8 常见配置示例

**例 1：两个 ChatGPT 账号，自动切换**

1. 供应商 → ChatGPT → 再点一次卡片，登录第二个账号；
2. 展开 ChatGPT 那一行，勾选两个账号；
3. 路由选 `smart`（默认）；
4. Agent 正常选 `codex/…` 模型即可。

**例 2：Claude 订阅 + Copilot 都有 Opus**

- magpie 会自动生成 `auto/claude-opus-…`；
- 给 Agent 选这个组；
- 请求会在两个来源之间分摊，额度用完自动换。

**例 3：省钱优先**

```sh
magpie group add "Cheap first" \
  models=deepseek/deepseek-v4-flash,kimi/kimi-k2.5 \
  routing=order
```

**例 4：长对话用强模型，短对话用便宜模型**

```sh
magpie group add "Daily" models=deepseek/deepseek-v4-flash,claude/claude-opus-5-5 routing=order
magpie group rule add daily use=claude/claude-opus-5-5 tokens=150k
```

**例 5：工作时间不用贵的账号**

```sh
magpie group rule add daily pause=claude/claude-opus-5-5 time=09:00-18:00 days=mon-fri
```

### 9.9 验证路由

- **路由** 标签页实时显示每个请求由谁回答、为什么；
- **用量** 页按 provider / 账号统计；
- `magpie groups` 列出你建的组和自动发现的组；
- `magpie group <id>` 查看某个组的成员和顺序。

---

## 10. 用量、花费与额度

### 10.1 用量页

**用量** 标签页按 Agent 和模型统计每次经过网关的调用：

- token 数；
- 缓存读写；
- 推理 token；
- 调用次数；
- 估算花费（美元或人民币）；
- 可选 **今天 / 7 天 / 30 天 / 全部**。

顶部是各订阅已用的额度和重置时间。

> **只统计经过 magpie 的请求。** Agent 直接发给自己厂商的请求不经过网关，不会计入。

### 10.2 上下文分析

**用量 → 上下文** 标签页显示每个 Agent 的提示词里装了什么：

- 系统提示词；
- 工具定义（一个 MCP 服务器的工具算一项）；
- 指令文件与 skills；
- 工具读入的文件；
- 工具的其他输出；
- 对话。

每项占模型上下文窗口的百分比。每个 Agent 还有一个百分制得分（缓存命中、起步轻重、上下文增长速度、失败率）和"起步轻""MCP 偏重"之类的标签。

> 原文只为**最近 10 次请求**留在内存里，magpie 退出就没了；存下来的只有名称，不保存提示词文字，也不会发往任何地方。

### 10.3 额度

```sh
magpie quota                  # 每个套餐还剩多少
magpie quota refresh codex    # 现在重新读取 Codex 的额度卡片（最多每 30 秒一次）
magpie quota wait codex       # 阻塞，直到某个 Codex 账号恢复额度
```

额度卡片包括：

- 每个 key、套餐和账号的余额；
- 额度窗口（5 小时 / 周 / 月）；
- 重置时间；
- 窗口还剩很多就要重置时提醒你。

### 10.4 会话

**用量 → 会话** 显示每个 Agent 对话的花费和标题，可以一键在终端里重新打开。

### 10.5 命令行用量

```sh
magpie usage                  # 默认
magpie usage today
magpie usage 7d
magpie usage 30d
magpie usage all
magpie usage --last 24h
magpie usage --days 1.5
magpie usage --csv
```

CSV 适合导入表格做自己的统计。

### 10.6 价格

- 默认按官方价格计算；
- 任何模型都可以自定义价格；
- 每个 provider 可以设置价格倍率；
- 部分 provider 有分时倍率（例如 DeepSeek 高峰时段双倍计价）；
- 两台电脑通过远程 magpie 共享时，计价在共享端完成，结果一致。

---

## 11. 隐私与本地脱敏

### 11.1 先分清两件事

| | 说明 | 默认 |
| :-- | :-- | :-- |
| **请求本身** | 你的 prompt / 回复只发给你选的供应商，不经过 magpie 的服务器 | 始终 |
| **本地脱敏** | 在请求离开电脑前，把密钥、密码、邮箱等换成占位符 | **关闭** |

> **只覆盖经过 magpie 的请求。** 接入 magpie 的 Agent 通过网关发请求，不管由哪个供应商或账号回答都会脱敏；绕过 magpie、直接请求厂商的 Agent 不在范围内。

### 11.2 打开本地脱敏

**设置 → 隐私**：

1. 在"要遮什么"下，把 **脱敏密钥** 设为开启：API Key、私钥、Token、密码；
2. 按需开启 **脱敏个人信息**：邮箱、手机号、身份证号、银行卡号等；
3. 在 **脱敏词** 里填不想让厂商看到的人名、项目代号、主机名，用逗号分隔；
4. 需要时打开"脱敏词不区分大小写"；
5. magpie 不认识的密钥（比如你自己网关的 key）加到 **自定义脱敏规则**：填开头或正则；
6. 用 **试一试** 粘贴一段文字，看厂商实际会收到什么。

改动从下一个请求开始生效，不用重启，Agent 也不用改。

> 实际做脱敏的是内置插件 **隐私遮盖**（`redact`）。只要设置里要求了任何脱敏，它就是开着的。

### 11.3 效果示例

开启"脱敏密钥"和"脱敏个人信息"，并在脱敏词里填 `Nightjar`。

Agent 发出：

```
Deploy Nightjar with OPENAI_API_KEY=sk-proj-8fK2mQx7Lp4Rt9Vw3Yz6Ab1Cd, then mail li.wei@acme-corp.cn or call 13800138000.
```

厂商收到：

```
Deploy {{TERM_5hmsrhdc}} with OPENAI_API_KEY={{API_KEY_srhjgvja}}, then mail {{EMAIL_eh5um6mb}} or call {{PHONE_uf7iy2qq}}.
```

模型回复里的占位符会换回原值——流式文本和工具调用参数里都算，所以 Agent 执行的命令里是真 key。

同一个值每次得到同一个占位符，所以对话上下文、提示词缓存和 thinking 签名都不受影响。原值**只保存在内存里**；磁盘上的记录只留每个值的类型、长度和开头几个字符。

### 11.4 能遮什么

**脱敏密钥：**

- 私钥块（`-----BEGIN … PRIVATE KEY-----`）；
- 按格式识别的 API Key：`sk-…`（OpenAI、Anthropic、OpenRouter）、`ghp_` / `github_pat_`、`glpat-`、`AIza…`、`xox…-`、`sk_live_`、`AKIA` / `ASIA`、`hf_`、`gsk_`、`xai-`、`npm_`、`pypi-`、`dop_v1_`、`SG.`，以及 `oc_sk_…` 这类中转站 key；
- JWT；
- `user:password@host` 里的密码；
- `.env` 和配置文件里以密钥命名的字段值：`DB_PASSWORD=…`、`api_key: …`、`client_secret`、`token`、`credential`；
- 你的自定义规则。

**脱敏个人信息：**

- 邮箱；
- 中国大陆手机号（11 位编号不算：`order_id` 下或"订单号"后面的照原样发）；
- 校验位对得上的身份证号和银行卡号；
- 默认也开着：美国社会安全号、护照号、IBAN、生日、MAC 地址、序列号；
- 默认关闭、需要手动打开：主目录用户名、`user@host`、公网 IP、云存储桶。

### 11.5 不能遮什么

官方明确列出的限制：

- **思考内容保持厂商原样**：带签名的 thinking 块、Gemini 的 thoughts、Responses 的加密推理，签名要对得上，里面只会替换请求其他地方已经遮掉的值。
- **ID 和工具定义原样发送**：模型 ID、消息和调用 ID、UUID、工具名、MCP 服务名、工具 schema（其中的描述文字会脱敏）。
- **图片和文件数据原样发送**：截图里的文字不会被遮。
- **模型只看到占位符**：它无法读懂或推理被遮掉的值，只能原样传递。需要模型读的值放进 **从不脱敏**。
- **规则按格式匹配**：其他国家格式的电话号码、自由文本里的人名，需要本地模型、脱敏词或自定义规则。

### 11.6 失败时的行为

- 脱敏已打开却运行不了（插件被手动关掉、本地模型没响应）→ 请求返回 **503**，说明什么都没发出；
- **绝不会**退回成不脱敏直接发送；
- 默认打开的本地模型没响应时，只用规则脱敏。

### 11.7 用本地模型查找

在 **怎么找** 下面：

| 模型 | 大小 | 说明 |
| :-- | :-- | :-- |
| **AstrLink Guard**（默认） | 约 39 MB | 中英文，代码里很少误判，Mac 上用神经网络引擎运行 |
| **OpenAI Privacy Filter** | 约 1.6 GB | 可用 GPU，代码里会遮盖更多普通值 |

- 除非你关掉，它默认打开；
- 第一个要脱敏的请求开始下载，下载完成前只用规则脱敏；
- 需要先打开"脱敏密钥"或"脱敏个人信息"；
- 置信度阈值：Privacy Filter 默认 0.5，AstrLink Guard 默认 0.8（发布者的值）。

### 11.8 按 provider 跳过

可以给某个 provider 打开 **请求不脱敏**，它的请求就原样发出：

- 本机或局域网的模型（Ollama、LM Studio）；
- 你选择的其他 provider（订阅和插件提供的也可以）；
- 命令行：`magpie provider set ollama unmasked=yes`。

> 对不在本地的 provider 打开时，编辑器会提示：请求会不脱敏地发给它的服务商。

### 11.9 确认生效

- **事前**：设置 → 隐私 → **试一试**；
- **每个请求**：网关页的最近调用和路由日志里，被脱敏的请求带盾牌徽标和数字；悬停看类型，点开看记录；
- **一段时间**：**观测 → 脱敏**（隐私页的"脱敏记录"也能进）按类型、Agent、provider 显示 7 天或 30 天里遮掉的值，以及每个请求在厂商眼里的原文。

记录保存在 `~/.config/magpie/masking`，保留 30 天：Agent、provider、模型，以及每个占位符的类型、长度和开头几个字符，**从不保存原值**。

### 11.10 遥测：magpie 自己会发什么

正式版 magpie 每天向 PostHog 发送一次"它在用"的事件：

- 本机随机生成的 ID（32 位十六进制，与你和这台电脑无关）；
- magpie 版本、`kind`（`app` 或 `serve`）、系统、架构；
- 用了几个 agent、provider、模型、路由组（关掉"分享我用的"后不发）；
- 每个 agent / provider / 模型各一个事件，只用 magpie 自己的 ID；你自己添加的 provider 只记作 `custom`。

**不发送**：名称、Base URL、账号、密钥、提示词、回复、用量。

**怎么关：**

- 设置 → 隐私 → **计入用户数**；
- 或 `settings.json` 里 `"noStats": true`；
- 或环境变量 `DO_NOT_TRACK=1` / `MAGPIE_NO_STATS=1`（除空、`0`、`false` 外的任何值）；
- 只关"分享我用的 agent、provider 和模型"：`"noUsageStats": true`；
- 关自动更新：设置 → 关于 → 自动更新，或 `"noAutoUpdate": true`。

> 从源码构建的版本从不发送。

### 11.11 请求存档

**用量页请求列表上方的"请求存档"开关默认关闭。** 打开后，每次经过网关的调用的请求头和正文会存到 magpie 文件夹里的 `request-archive`：

- 一次调用一个文件；
- 只有你的用户能读；
- 密钥按 S3 存档同样的方式遮掉；
- 最多占 1 GiB（`settings.json` 里的 `requestArchiveLocalMB`），超过后先删最早的；
- 随时可以删。

---

## 12. 命令行速查

`magpie help` 列出全部命令。`magpie tui` 是终端里的完整界面。

### 12.1 查看状态

| 命令 | 作用 |
| :-- | :-- |
| `magpie ls` | 列出找到的每个 Agent 及其设置 |
| `magpie providers` | 你的 provider：地址、密钥、模型、谁在用 |
| `magpie models` | Agent 能选的所有模型，写作 `provider/model` |
| `magpie presets` | magpie 内置的厂商预设 |
| `magpie quota` | 每个套餐还剩多少 |
| `magpie usage 7d` | 按 Agent 和模型统计 token 与费用 |
| `magpie settings` | 所有设置及其值（`--json` 也可） |
| `magpie log` | 日志路径；`-n 200` 最后 200 行；`-f` 跟随 |

### 12.2 Provider 管理

| 命令 | 作用 |
| :-- | :-- |
| `magpie provider add deepseek sk-…` | 用 key 添加预设 |
| `magpie provider add ollama` | 添加本地 provider |
| `magpie provider add "My Relay" url=… key=… models=…` | 添加自定义 provider |
| `magpie provider test deepseek` | 每个 API 发一个最小请求，带延迟 |
| `magpie provider models deepseek` | 重新拉取模型列表 |
| `magpie provider models deepseek +model -model` | 增加/移除暴露的模型 |
| `magpie provider refresh deepseek` | 重新拉取并丢弃已不存在的选择 |
| `magpie provider key deepseek sk-…` | 替换 key |
| `magpie provider rm deepseek` | 删除 |
| `magpie providers` | 查看全部 |

### 12.3 Agent 管理

| 命令 | 作用 |
| :-- | :-- |
| `magpie claude deepseek/deepseek-v4-pro` | 设置 Claude Code 的模型 |
| `magpie codex moonshot/kimi-k2.5` | 设置 Codex 的模型 |
| `magpie opencode anthropic/claude-sonnet-5` | 设置 OpenCode 的模型 |
| `magpie codex effort high` | 设置推理强度 |
| `magpie codex effort` | 查看当前字段 |
| `magpie claude haiku deepseek/deepseek-v4-flash` | 单独设置某档 |
| `magpie claude haiku ""` | 恢复跟随主模型 |
| `magpie <agent> default` | 恢复 Agent 自己的默认值，移除 magpie 的接入 |

Agent 名字支持前缀：`cc`、`oc`、`gem` 等。

### 12.4 方案

| 命令 | 作用 |
| :-- | :-- |
| `magpie save work` | 保存当前配置为方案 |
| `magpie use work` | 应用方案 |
| `magpie profiles` | 列出方案 |
| `magpie rm work` | 删除方案 |

### 12.5 路由组

| 命令 | 作用 |
| :-- | :-- |
| `magpie groups` | 列出你建的组和自动发现的组 |
| `magpie group add "名字" models=… routing=…` | 新建组 |
| `magpie group set <id> models+=…` | 增加成员 |
| `magpie group set <id> routing=weight weight=…` | 设置路由和权重 |
| `magpie group rm <id>` | 删除；自动组是隐藏，可恢复 |
| `magpie group restore <id>` | 恢复自动组 |
| `magpie group copy <id> "新名字"` | 复制 |
| `magpie group rule add <id> use=… tokens=…` | 添加规则 |
| `magpie group rule <id>` | 查看规则 |
| `magpie claude group/<id>` | 让 Agent 使用组 |

### 12.6 账号与网关

| 命令 | 作用 |
| :-- | :-- |
| `magpie accounts add codex` | 再登录一个 Claude 或 ChatGPT 订阅 |
| `magpie accounts switch claude next` | 切到下一个还有额度的账号 |
| `magpie accounts discovery off` | 不读取智能体登录信息；`on` 恢复 |
| `magpie gateway-key add "Laptop"` | 给其他电脑生成网关密钥 |
| `magpie gateway-key limit …` | 给密钥设限额 |
| `magpie gateway-key models …` | 给密钥限定可用模型 |

### 12.7 运行方式

| 命令 | 作用 |
| :-- | :-- |
| `magpie` | 打开应用：窗口 + 菜单栏图标 |
| `magpie tray` | 只启动菜单栏图标 |
| `magpie tui` | 终端里的完整界面 |
| `magpie web` | 浏览器界面（WSL / SSH 服务器） |
| `magpie serve` | 只运行网关 |
| `magpie daemon install` | 装成后台服务（systemd / launchd） |
| `magpie daemon status` | 运行状态、pid、文件、日志、网关是否应答 |
| `magpie daemon restart` | 更新后重启 |
| `magpie daemon stop` / `start` | 停止 / 启动 |
| `magpie daemon uninstall` | 卸载服务 |
| `magpie autostart on` | 登录时自动打开（Windows） |
| `magpie healthcheck` | 网关能应答时退出码 0 |

### 12.8 工具与维护

| 命令 | 作用 |
| :-- | :-- |
| `magpie tools` | 列出 MCP 工具及参数 |
| `magpie tool generate_image "…" -o hero.png` | 直接调用生图工具 |
| `magpie tool analyze_image ./shot.png --prompt "…"` | 识图 |
| `magpie tool web_search "…"` | 联网搜索 |
| `magpie settings get/set/reset …` | 读写设置 |
| `magpie sync` | 刷新 models.dev 目录和模型列表 |
| `magpie backup` | 备份（加密，询问口令） |
| `magpie restore <file>` | 恢复 |
| `magpie update` | 更新 |
| `magpie update channel stable` | 切到稳定版通道 |
| `magpie update mirror <前缀>` | 设置更新镜像；`off` 取消 |
| `magpie uninstall` | 断开所有 Agent 并删除 magpie 数据 |
| `magpie webdav on …` | WebDAV 同步 |
| `magpie s3 on …` | S3 同步 |
| `magpie github on …` | GitHub 同步 |

---

## 13. 文件、日志与配置目录

### 13.1 配置目录

magpie 的文件夹是：

| 系统 | 路径 |
| :-- | :-- |
| macOS | `~/.config/magpie` |
| Linux | `~/.config/magpie`（尊重 `$XDG_CONFIG_HOME`） |
| Windows | `%USERPROFILE%\.config\magpie` |

设置 `$XDG_CONFIG_HOME` 时是 `$XDG_CONFIG_HOME/magpie`。

### 13.2 目录里有什么

**你设置的：**

| 文件 | 内容 |
| :-- | :-- |
| `settings.json` | 设置 |
| `providers.json` | provider 和 key（`0600`） |
| `profiles.json` | 方案 |
| `library.json`、`library/` | 资料库：指令、skills、MCP servers、hooks |
| `plugins.json`、`plugins/` | 插件及其包 |
| `plugins.yml` | 内置插件开关和配置 |
| `caller-keys.json` | 给其他调用方生成的网关密钥（`0600`） |

**账号与登录：**

| 文件 | 内容 |
| :-- | :-- |
| `logins.json` | 通过 magpie 登录的订阅账号（`0600`） |
| `plugin-auth.json` | 插件的登录（`0600`） |
| `mcp-signins.json` | MCP server 的登录（`0600`） |
| `cli-identity.json` | Agent 自己的 CLI 最后报告是谁登录 |
| `redact.key` | 脱敏占位符背后的密钥 |

**magpie 写进 Agent 的内容：**

| 文件 | 内容 |
| :-- | :-- |
| `stash.json` | 被 magpie 替换的值，切回时恢复 |
| `applied.json` | magpie 最后给每个 Agent 设置的内容和时间 |
| `migrations.json` | 订阅迁移到插件的情况 |

**网关与用量：**

| 文件 | 内容 |
| :-- | :-- |
| `usage.jsonl` | 每个请求一行，用量页的数字 |
| `routing/` | 每天的请求路由记录（`<date>.jsonl`） |
| `affinity.json` | 哪个 key/账号回答过哪个对话 |
| `served.json` | 账号和 key 最后回答的时间 |
| `quotas.json` | 账号最后报告的额度 |
| `quota-history.json`、`balance-history.json` | 额度和余额的历史 |
| `usage-alerts.json` | 已经显示过的用量提醒 |
| `client-shapes.json` | 网关最后放行的各客户端请求头 |
| `gateway-sessions-hidden.json` | 你删除的网关会话 |

**缓存目录 `~/.cache/magpie`：**

| 文件 | 内容 |
| :-- | :-- |
| `bun/<version>/magpie-bun` | 插件运行的 Bun |
| `models.json` | models.dev 目录 |
| `models/<provider>.json` | 从厂商拉取的模型列表 |
| `partner-counts.json` | 未发送的合作伙伴计数（最多 14 天） |

### 13.3 日志

| 系统 | 应用日志 |
| :-- | :-- |
| macOS | `~/Library/Logs/magpie/app.log` |
| Windows | `%LOCALAPPDATA%\magpie\logs\app.log` |
| Linux | `~/.local/state/magpie/app.log`（尊重 `$XDG_STATE_HOME`） |
| 便携版 | 数据文件夹的 `logs/` |

服务模式的日志：

| 系统 | 位置 |
| :-- | :-- |
| macOS | `~/Library/Logs/magpie/serve.log` |
| Linux | `journalctl --user -u magpie` |

日志满 5 MiB 后变成 `app.old.log`，最多保留两个文件。日志不含密钥、提示词或回复；看起来像密钥的内容也会被遮掉。

```sh
magpie log          # 打印路径
magpie log -n 200   # 最后 200 行
magpie log -f       # 持续跟随
```

---

## 14. 进阶功能（按需阅读）

### 14.1 插件

插件负责登录某个服务、替它发请求。magpie 把插件变成一个 provider，所有 Agent 都能用，多账号、故障切换、用量统计也都有。

**安装插件：**

```sh
magpie plugin add opencode-gemini-auth   # npm 上的 OpenCode 插件
magpie plugin add pi-antigravity         # pi 包
magpie plugin login google-plugin        # 走插件自己的登录流程
```

插件运行在 Bun 上，第一次需要时自动下载。

**插件能做什么：**

- 登录（API key、浏览器、设备码）；
- 列出模型；
- 发送请求；
- 续期登录；
- 每日签到；
- 报告套餐用量。

**网关中间件：**

插件也可以是一段 JavaScript，能看到每个 Agent 发出和收到的内容：

- `onRequest`：改写或拒绝请求；
- `onEvent`：看到每个流式事件；
- `onResponse`：看到完整回复；
- 钩子抛错或超时，请求原样放行。

```js
// alias.middleware.js
export function onRequest(body, ctx) {
  if (body.model === "fast") return { ...body, model: "deepseek/deepseek-chat" };
}
```

```sh
magpie plugin add ./alias.middleware.js
```

**现成的中间件：**

| 包 | 作用 |
| :-- | :-- |
| `param-override` | 按条件设置、删除、移动或改写请求字段，或拒绝请求 |
| `model-map` | 把模型换个名字发出去，回复里仍是原来请求的名字 |
| `system-prompt` | 给所有请求，或某些 Agent、某些模型加上系统提示词 |
| `word-guard` | 敏感词过滤：在用户发送的内容和回复里拒绝或屏蔽 |
| `think-tags` | 去掉 `<think>…</think>`，或把 `reasoning_content` 放进回复 |

```sh
magpie plugin add @magpie-community/middleware-model-map
magpie plugin options model-map '{"mapping": {"fast": "deepseek/deepseek-chat"}}'
```

### 14.2 MCP、搜索与生图

**任何模型都能联网搜索：**

- 不能搜索的模型，由 magpie 代为搜索；
- 用能搜索的 provider 或搜索 API：Tavily、Brave、Exa、Firecrawl、You.com、DeepSeek、火山引擎豆包搜索、腾讯云元宝搜索、SearXNG；
- 按你拖好的顺序调用；
- 每个都能单独关闭而不删除密钥；
- 一个 API 可以填多个密钥，默认轮流使用（`pick=order` 则按顺序），最近被拒的密钥最后才用。

**给 Agent 挂 MCP：**

- 任何 Agent 都能把同一个搜索挂成 MCP server；
- 提供 `web_search` 工具和按 URL 读取网页的 `web_fetch` 工具；
- MCP router 可以把资料库里的多个 MCP server 合成一个 `magpie` 入口。

**生图 / 生视频：**

```sh
magpie tools
magpie tool generate_image "一只喜鹊，水彩" -o hero.png
magpie tool generate_image --n 2 --quality high --background transparent "an app icon"
magpie tool analyze_image ./shot.png --prompt "what does the error say?"
```

- 通过 MCP 给任何 Agent 一个生图工具；
- 用你选的模型；
- 有 Grok 订阅或 MiniMax API Key（MiniMax H3）还能生成视频；
- 生成的图片保存到 `-o`（文件或文件夹）或当前目录，不会覆盖已有文件。

### 14.3 远程 magpie：另一台电脑用同一套模型

一个 magpie 可以服务多台电脑。在保存 provider 的电脑上配置一次，其他电脑把它添加为 **远程 magpie**。

**第 1 步：在共享端打开共享**

- 设置 → 网络与共享 → **局域网共享** → 开启；
- magpie 随即在局域网监听，并生成一个名为 **Magpie** 的网关密钥；
- 开关下方显示其他电脑访问它的地址，例如 `http://192.168.1.20:3425`；
- 给每台电脑一个密钥：网关 → 网关密钥，或 `magpie gateway-key add "Laptop"`。

**第 2 步：在其他电脑上添加**

应用：

- 供应商 → ＋ 添加供应商 → 最下面的 **另一台电脑上的 magpie** → **Remote magpie**；
- 填地址和密钥。

TUI：

- `magpie tui` → Providers 页按 `a` → Remote magpie → 填地址和密钥；
- 在它那一行按 `w` 改地址，按 `m` 重新获取列表。

CLI：

```sh
magpie provider add remote-magpie sk-magpie-key-… url=http://192.168.1.20:3425 id=office
magpie models
magpie provider refresh office
```

`id=` 是本机的名字，也是模型 id 的前缀（`office/…`）。

**能用到什么：**

- 共享端的模型：`office/deepseek/deepseek-v4-pro`；
- 共享端的路由组：`office/group/…`；
- 所有 API 原样转发；
- 图像和视频模型；
- 额度卡片（带远程名字，如 `Codex · office`）；
- 用量在共享端统计，在本机显示。

**注意：**

- 本机的 Agent 仍由本机 magpie 配置；
- 只有模型来自另一台；
- 共享端不可用时，使用它的请求会失败；只有你的路由组里还放了本机模型时，才会由路由组接上。

### 14.4 Docker 与 NAS

**镜像：**

```
ghcr.io/yetone/magpie
```

支持 `linux/amd64` 和 `linux/arm64`。镜像以非 root 用户（uid 65532）运行；绑定挂载 `/config` 时要让它可写。

**标签：**

| 标签 | 含义 |
| :-- | :-- |
| `latest` | 最新正式版 |
| `stable` | 与桌面版稳定版通道一致：每周一个已发布满三天的版本 |
| `0.1.1150` | 某个具体版本 |
| `0.1` | 某个次版本 |
| `edge` | `main` 上的每次改动 |

**端口：**

| 端口 | 用途 |
| :-- | :-- |
| `3425` | Agent 调用的网关 |
| `3430` | 容器运行 `magpie web` 时的浏览器界面；也响应网关的 `/v1` 路由 |

**数据卷：**

```
/config
```

里面包括：

| 路径 | 内容 |
| :-- | :-- |
| `/config/magpie` | magpie 自己的文件：settings、provider、网关密钥、路由组 |
| `/config/home` | 容器的 `HOME`：登录保存在对应 Agent 的位置（`~/.codex`、`~/.claude`……） |
| `/config/cache` | 模型目录、插件用的 Bun、`magpie claude-code install` 下载的 Claude Code |
| `/config/data`、`/config/state` | 其他内容 |

**运行（带 Web 界面）：**

```sh
docker run -d --name magpie --restart unless-stopped \
  -p 127.0.0.1:3425:3425 -p 127.0.0.1:3430:3430 \
  -e MAGPIE_WEB_KEY=$(openssl rand -hex 16) \
  -v magpie-config:/config \
  ghcr.io/yetone/magpie:latest web --addr 0.0.0.0:3430 --no-open

docker logs magpie
```

日志里会给出带 key 的页面地址：

```
● magpie web on http://127.0.0.1:3430/?k=…
```

> 容器里的页面是**网关模式**：只有供应商、网关、路由、用量、插件和设置，没有 Agent、会话和资源库，因为容器里没有 Agent。用它的 Agent 运行在别的机器上。

**网关密钥：**

- 容器里任何 key 都行；
- 容器外的请求（包括宿主机自己的）必须带一个启用的网关密钥，否则 401；
- 生成：

```sh
docker exec magpie magpie gateway-key add "Laptop"
```

- 每个密钥可以有自己的每日/每周/每月上限和可用模型。

**环境变量：**

| 变量 | 作用 |
| :-- | :-- |
| `MAGPIE_ADDR` | 网关监听地址；镜像里是 `0.0.0.0:3425` |
| `MAGPIE_PUBLIC_URL` | 其他机器访问网关用的地址，例如 `http://192.168.1.20:3425` |
| `MAGPIE_WEB_KEY` | Web 界面 key，重启后不变（16 个字符以上） |
| `MAGPIE_TLS_CERT`、`MAGPIE_TLS_KEY` | 挂载进容器的 PEM 证书和私钥；其他机器只能用 https |
| `MAGPIE_ALLOW_IPS` | 只允许这些 IP/CIDR 访问；其他地址即使有密钥也 403 |
| `HOME`、`XDG_*` | 镜像已设为 `/config` 下的目录，不要改 |

**在容器里登录订阅：**

厂商登录完成后会把浏览器跳回 `localhost`（ChatGPT 是 `http://localhost:1455/auth/callback?code=…`），那是你自己的电脑，不是容器，所以页面打不开。把地址栏里的完整地址复制下来，粘贴到登录的 **回调 URL** 一栏。

终端方式：

```sh
docker exec -it magpie magpie accounts add codex
```

打开它打印的链接并登录，再把浏览器最后停在的地址粘贴回去。

**Claude 订阅：**

- 登录和请求都经过 Claude Code，而镜像里没有它；
- magpie 会说明版本、下载来源和保存位置，问过你再下载，并用 Anthropic 清单里的 SHA-256 校验；
- 保存在 `/config/cache`，换新镜像也还在；
- 缺少时也可以手动装：

```sh
docker exec -it magpie magpie claude-code install
```

**Grok 订阅：**

- 通过 xAI 的 Grok Build CLI 登录，镜像里同样没有它；
- 在浏览器界面登录，或 `magpie plugin login grok` 时，magpie 会先按 xAI 安装脚本的方式安装它。

**Compose 示例：**

```yaml
services:
  magpie:
    image: ghcr.io/yetone/magpie:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:3425:3425"
      - "127.0.0.1:3430:3430"
    environment:
      MAGPIE_PUBLIC_URL: http://<nas-address>:3425
      MAGPIE_WEB_KEY: <random-key>
    volumes:
      - ./config:/config
    command: [web, --addr, 0.0.0.0:3430, --no-open]
```

```sh
sudo chown -R 65532:65532 ./config
docker compose up -d
```

然后通过 SSH 隧道打开：

```sh
ssh -L 3430:127.0.0.1:3430 you@nas
```

在浏览器打开日志里的链接。

**安全提醒：**

- 拿到 Web 链接的人都能修改 magpie、看到它的密钥；
- 让 3430 只在回环地址或 VPN 上；
- 跨网络访问用 HTTPS（`MAGPIE_TLS_CERT` / `MAGPIE_TLS_KEY`）或带 TLS 的反向代理；
- 跨互联网优先用 Tailscale / WireGuard，不要直接开放 3425。

**更新：**

```sh
docker pull ghcr.io/yetone/magpie:latest
docker rm -f magpie
# 再执行同样的 docker run
```

或：

```sh
docker compose pull && docker compose up -d
```

**健康检查：**

镜像带 `HEALTHCHECK`：网关能应答时 `magpie healthcheck` 返回 0，`docker ps` 会显示 healthy。

### 14.5 备份与多机同步

**备份：**

```sh
magpie backup                            # 写 magpie.magpie-backup，询问口令两次
magpie backup --no-keys ~/b.magpie-backup  # 不含 API key
magpie restore magpie.magpie-backup        # 在另一台机器恢复
magpie restore --no-agents b.magpie-backup # 只恢复 provider、设置、方案
magpie restore --no-library b.magpie-backup
```

备份包含：

- provider（含 key，除非 `--no-keys`）；
- provider 的图标；
- 设置；
- 方案；
- 每个 Agent 的模型；
- 资料库（除非 `--no-library`）。

**不包含**：订阅登录（每台机器重新登录）。

加密：AES-256-GCM，密钥由口令经 PBKDF2-SHA256 派生；没有口令读不出内容。

**同步：**

设置 → 同步和备份 → WebDAV / S3 / GitHub，每 3 分钟同步一次。

```sh
magpie webdav on https://dav.jianguoyun.com/dav/ user=me@example.com
magpie s3 on s3://my-bucket/magpie endpoint=https://<account>.r2.cloudflarestorage.com access-key-id=…
magpie github on github://owner/private-backups/settings branch=main
```

- 文件在本地加密，服务器只存密文；
- 每台机器只覆盖它读到的版本（条件写），不会丢失另一台机器的更新；
- GitHub 仓库必须是私有的，token 需要 Contents 读写权限。

### 14.6 OTLP 导出

设置 → 可观测性 可以把网关请求元数据导出到你的 OpenTelemetry 收集器（如 Langfuse）：

```sh
MAGPIE_OTEL_ENABLED=true MAGPIE_OTEL_ENDPOINT=http://localhost:4318 magpie serve
```

- 默认关闭；
- 只发到你设置的地址；
- 可导出 agent、provider、模型、token 数、状态、耗时、估算费用；
- 另有开关加指标、本地 agent 对话、请求和回复正文（密钥已遮盖）。

### 14.7 后台服务

```sh
magpie daemon install    # 写服务并启动
magpie daemon status     # 运行状态、pid、文件、日志、网关是否应答
magpie daemon restart    # 更新后重启
magpie daemon stop
magpie daemon start
magpie daemon uninstall  # 停止并删除服务文件
magpie daemon web on     # 服务同时提供浏览器页面
```

- **Linux**：systemd user unit `~/.config/systemd/user/magpie.service`，用 `systemctl --user`；日志 `journalctl --user -u magpie -f`；
- **macOS**：launch agent `~/Library/LaunchAgents/com.yetone.magpie.serve.plist`，日志 `~/Library/Logs/magpie/serve.log`；
- **Windows**：不是服务；用 `magpie autostart on`，登录时打开 magpie，由它提供网关；
- **WSL**：和 Linux 一样；需要先打开 systemd（`/etc/wsl.conf` 里 `[boot] systemd=true`）。

> 服务模式和 `magpie autostart` 不能同时用：两者会抢同一个端口。安装服务时如果 autostart 开着，会被拒绝。

### 14.8 资料库与 Agent 版本管理

**资料库：**

- 指令、MCP servers、skills、子代理只写一次；
- magpie 按每个 Agent 的格式写进它的文件；
- 删除时只删自己写的部分。

适合多 Agent 共用同一套规则和工具。

**Agent 版本管理：**

- **Agent** 页显示每个 CLI 的版本；
- 按当初的安装方式更新；
- 没装的 Agent 也会给出安装命令。

---

## 15. 常见问题 FAQ

### Q1：需要自己 export `OPENAI_BASE_URL` 或 `ANTHROPIC_BASE_URL` 吗？

**不需要。** 对 magpie 列表里的 Agent，它会把网关地址和令牌直接写进 Agent 自己的配置文件。

这些环境变量只用于 magpie 不管理的、带 base URL 设置的其他工具。

### Q2：magpie 必须一直开着吗？

通过 magpie 使用的模型需要网关在运行。

- 关掉窗口后它仍留在菜单栏 / 托盘；
- `magpie tray` 只启动图标（可以放进登录项）；
- `magpie serve` 只运行网关；
- `magpie daemon install` 把它装成服务（Linux 是 systemd，Mac 是 launchd），登录时启动，不需要窗口；Linux 上无人登录也要运行需 `sudo loginctl enable-linger $USER`。

Agent 用它自己的模型和自己的登录时，不经过 magpie。

### Q3：怎么把 Agent 恢复原样？

- 在模型选择器里选 Agent 自带的模型；或
- 运行 `magpie <agent> default`。

magpie 会删掉它写入的内容，并恢复被它替换掉的值（比如你原来的 `ANTHROPIC_BASE_URL`）。

### Q4：换了模型，Agent 还在用旧的？

已在运行的会话保持启动时的模型。**开一个新会话**；如果是 Codex，还要**重启 Codex 应用**和已打开的 `codex` 会话。

### Q5：手动改 Agent 的配置会冲突吗？

不会。magpie 每次都重新读取文件，只动它设置的那几个键，你的其他设置和注释都会保留。

用"导入…"带进来的 provider 是一份拷贝：之后在 Agent 里的改动不会再同步过来。

### Q6：没有梯子，安装或更新连不上 GitHub 怎么办？

```sh
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --proxy http://127.0.0.1:7890
magpie update --proxy socks5://127.0.0.1:1080
```

支持 http、https、socks5、socks5h；`HTTPS_PROXY`、`ALL_PROXY` 同样生效。

也可以用镜像：

```sh
magpie update --mirror https://mirror.example/
magpie update mirror <前缀>    # 之后每次更新都用它
magpie update mirror off       # 取消
```

### Q7：Linux 上安装脚本装的是哪个版本？

- 有 GTK 3 + WebKitGTK 4.1 → 桌面版；
- 有桌面但缺库 → 脚本用包管理器装 WebKitGTK 4.1，再装桌面版；
- 没有终端、没有 sudo、发行版不认识或装库失败 → 终端版，并告诉你该装哪个包；
- 想自己选：`--gui` 或 `--tui`。

### Q8：怎么卸载 magpie？

```sh
# 脚本安装的：卸载程序，保留设置
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --uninstall

# 彻底删除：断开 Agent、恢复原配置、删除设置/provider/用量
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --uninstall --purge
```

用其他方式安装的：

```sh
magpie uninstall
```

或设置 → 卸载 magpie。

### Q9：magpie 的日志在哪里？

| 系统 | 路径 |
| :-- | :-- |
| macOS | `~/Library/Logs/magpie/app.log` |
| Windows | `%LOCALAPPDATA%\magpie\logs\app.log` |
| Linux | `~/.local/state/magpie/app.log` |

`magpie log` 打印路径，`magpie log -f` 持续跟随。

### Q10：magpie 会发送我的哪些信息？

- 你的 prompt、回复、密钥、账号**只发给你用的供应商**，中间没有 magpie 自己的服务器；
- 正式版每天发一次匿名计数（随机 ID、版本、系统、用了哪些 agent/provider/model）；
- 不发送名称、Base URL、账号、密钥、prompt、用量；
- 关闭方法见 [11.10](#1110-遥测magpie-自己会发什么)。

### Q11：Agent 报 502 或连不上，但路由页什么都没有？

代理软件把发往 `127.0.0.1:3425` 的请求接走了。解决办法见 [16.2](#162-agent-报-502但路由页什么都没有)。

### Q12：Claude Code 里能用 ChatGPT 订阅吗？

可以。在 magpie 里登录 ChatGPT 订阅，然后在 Claude Code 那一行选 `codex/…` 模型。

### Q13：Codex 里能用 Claude 订阅吗？

可以。登录 Claude 订阅，在 Codex 那一行选 `claude/…` 模型。

> Claude 订阅的请求通过本机 Claude Code 运行，所以请保留 Claude Code。

### Q14：能在 Claude Code 里用 Antigravity 的模型吗？

可以。在 magpie 里登录 Antigravity，然后在 Claude Code 那一行选 `antigravity/…` 模型。

> Google 可能封禁在 Antigravity 之外使用的 Antigravity 账号，请用丢了也不心疼的账号。

### Q15：一个 provider 能放多个账号吗？

可以。再点一次同一张卡片即可再加一个账号；它们都列在这个 provider 下面，点一下切换，或交给路由自动分配。

### Q16：多个账号额度用完了会自动换吗？

会。默认的 `smart` 路由会在还有余量的订阅里优先用最快重置的那个。只有一个账号又没建路由组时，没有可换的，报错会原样回给 Agent。

### Q17：magpie 支持本地模型吗？

支持。Ollama 和 LM Studio 都在预设里，无需 key。

### Q18：我的中转站没有预设怎么办？

用 **自定义**：填 OpenAI 兼容地址（以 `/v1` 结尾）或 Anthropic 兼容根地址，再填 key 和模型列表。

### Q19：magpie 会改坏我的配置吗？

不会。它只改需要的键，保留注释、顺序、缩进，写入是原子的；`magpie <agent> default` 可以恢复。

### Q20：换电脑怎么迁移？

用 `magpie backup` / `magpie restore`，或 WebDAV / S3 / GitHub 同步。订阅登录不包含在内，每台机器要重新登录。

### Q21：magpie 收费吗？

magpie 本身是 **MIT 许可、免费**的。你要付的是各 provider 的模型费用或订阅费用，按 provider 自己的计费方式结算。

### Q22：DeepSeek Harness 能保留自己的 DeepSeek 行，同时走 magpie 吗？

可以，需要手动配置。连接 dsh 时 magpie 只加自己的路由，dsh 的 DeepSeek 行（`llm-deepseek`）留给 dsh。要让它也经过 magpie，在 `~/.dsh/profiles/<profile>/cordis.patch.yml` 里给这一行加上：

```yaml
- id: llm-deepseek
  config:
    baseURL: http://127.0.0.1:3425
    apiKeyEnv: MAGPIE_DS_KEY
    models:
      - id: deepseek/deepseek-flash
        name: DeepSeek V4.1 Flash (magpie)
        contextWindow: 1000000
        inputModalities: [text, image]
        systemPromptUpdate: in-history
        toolUpdate: addition-only
```

- `baseURL` 不带 `/anthropic`；
- `MAGPIE_DS_KEY` 里放**网关密钥**（本机是 `magpie`），不是你的 DeepSeek key；
- 模型 id 用 `magpie models` 的输出；
- 详见官方参考文档的 DeepSeek Harness 一节。

---

## 16. 故障排查手册

### 16.1 先做这三件事

```sh
magpie ls                    # Agent 当前设置
magpie provider test <id>    # provider 是否可用
magpie log -n 200            # 最近的错误
```

90% 的问题能在这里定位。

### 16.2 Agent 报 502，但路由页什么都没有

**症状：**

- Agent 报 502 / 连不上；
- magpie 的网关页或路由页看不到任何请求。

**原因：**

代理软件（Surge、Clash、mihomo 等，开了系统代理、TUN 或增强模式）把 Agent 发往 `http://127.0.0.1:3425` 的请求接走、转去了别处，请求根本没到 magpie。

只在跳过列表里写 `127.0.0.1` 不一定够：有些 Agent（比如 Codex）照样会走代理。

**解决：**

在代理软件里加两条本地地址直连的规则，放在其他规则前面。

Surge：

```
IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
IP-CIDR6,::1/128,DIRECT,no-resolve
```

Clash / mihomo：写在 `rules:` 下，每行前面加 `- `：

```yaml
rules:
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR6,::1/128,DIRECT,no-resolve
```

改完让 Agent 重新发一次请求即可。

### 16.3 Docker 里返回 401

**原因：** 容器外的请求必须带一个启用的网关密钥。

**解决：**

```sh
docker exec magpie magpie gateway-key add "Laptop"
```

把打印出来的 `sk-magpie-key-…` 填进 Agent 的 API key 设置。

旧镜像升级后，原来发送 `magpie` 或其他非启用密钥的 Agent 会收到 401；给它换一个启用的网关密钥即可。

### 16.4 模型列表是空的 / 找不到某个模型

可能原因：

1. **provider 没拉取到模型**：`magpie provider models <id>` 重新拉；
2. **模型被隐藏**：展开 provider 那一行，检查暴露的模型；
3. **模型已弃用**：厂商说 deprecated / retired 的模型会从列表里消失，直到有人用 `provider/model` 请求它，或过一周；
4. **模型 ID 写错**：用 `magpie models` 看准确写法；
5. **订阅没登录成功**：重新登录，或检查 `magpie accounts`；
6. **自定义 provider 没填 models**：在编辑页补上，或 `magpie provider models <id> a b c`。

### 16.5 provider 测试失败

按错误类型排查：

| 现象 | 先查 |
| :-- | :-- |
| 401 / 403 | key 是否正确、是否过期、是否有权限 |
| 404 | base URL 是否正确（OpenAI 兼容要以 `/v1` 结尾；Anthropic 兼容用根地址） |
| 429 | 额度 / 频率限制；换账号或等重置 |
| 超时 | 网络 / 代理 / 厂商服务状态 |
| TLS 错误 | 代理软件拦截、系统证书、base URL 协议 |
| 模型不存在 | 模型 ID 是否是该 provider 的真实 ID；用 `magpie models` 确认 |

### 16.6 改了模型但 Agent 还用旧的

- Agent 只在启动时读配置：**开新会话**；
- Codex 还要重启应用；
- Claude Desktop 需要重启；
- 编辑器插件要重新加载窗口；
- 确认改的是正确的 Agent 行（`magpie ls`）。

### 16.7 端口被占用 / 网关起不来

- 检查是否有另一个 magpie 在跑：`magpie daemon status`；
- 服务模式和 `magpie autostart` 不能同时用；
- 改了 `MAGPIE_ADDR` 时确认端口没被其他程序占用；
- 用 `magpie log -n 200` 看具体错误。

### 16.8 安装脚本失败

| 现象 | 处理 |
| :-- | :-- |
| 下载失败 / 超时 | `--proxy` 或 `--mirror` |
| Linux 缺 WebKitGTK | 按提示装 `libwebkit2gtk-4.1-0`（Debian/Ubuntu）或 `webkit2gtk4.1`（Fedora） |
| 没有桌面 | 用 `--tui` |
| 没有 sudo | 用 `--tui`，或手动装依赖 |
| Alpine | 只能用终端版 |
| `magpie: command not found` | 重开终端；确认安装脚本的输出；检查 `PATH` |

### 16.9 Web 界面打不开 / 401

- 链接必须带 `?k=<key>`；
- 没设 `MAGPIE_WEB_KEY` 时，每次启动 key 都会变，旧链接失效；
- 设置了 `MAGPIE_WEB_KEY` 时，浏览器保持登录 400 天；
- 反向代理场景注意 `MAGPIE_PUBLIC_URL` 和 `MAGPIE_WEB_URL`；
- 同一地址上的其他服务可能清掉 cookie 或占满 cookie 限额，导致登录丢失。

### 16.10 订阅登录回调失败（Docker / 远程）

厂商把浏览器跳回 `localhost`，但 `localhost` 是你自己的电脑，不是容器/远程主机。

**解决：** 复制浏览器地址栏里的完整回调地址，粘贴到登录界面的 **回调 URL** 一栏。

### 16.11 额度显示不对 / 不刷新

- 额度最多每 30 秒读一次：`magpie quota refresh codex`；
- 共享 magpie 的额度在共享端读取，本机显示的是那边最近一次的结果；
- 想完全手动刷新：设置 → 额度 → 额度刷新 → 仅手动。

### 16.12 用量里没有某些请求

只有**经过网关**的请求才会计入。以下情况不会计入：

- Agent 用它自己的模型、自己的登录，直连厂商；
- 你在 Agent 里临时切到了非 magpie 的 provider；
- 请求在到达网关前就被代理接走了（见 16.2）。

### 16.13 脱敏打开后请求返回 503

说明脱敏已打开但运行不了：

- 检查内置插件 **隐私遮盖** 是否被手动关掉；
- 如果开了本地模型，检查它是否在运行 / 下载完成；
- 这是有意设计：**绝不退回成不脱敏直接发送**；
- 临时处理：关掉脱敏，或把该 provider 设为"请求不脱敏"。

### 16.14 模型被厂商弃用

厂商说 deprecated / retired 的模型会从 `magpie models`、`/v1/models` 和 Agent 列表里消失，即使厂商自己的列表还列着它。

它会在有人用 `provider/model` 请求它之后回来，或过一周。

### 16.15 想彻底重来

1. `magpie backup` 先备份；
2. `magpie <agent> default` 逐个恢复 Agent；
3. 或 `magpie uninstall`（断开所有 Agent 并删除 magpie 数据）；
4. 重新安装、重新添加 provider。

---

## 17. 卸载与恢复原样

### 17.1 只恢复某个 Agent

```sh
magpie claude default
magpie codex default
```

或在模型选择器里选 Agent 自带的模型。

### 17.2 卸载脚本安装的内容（保留设置）

```sh
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --uninstall
```

会拿掉：

- `magpie` 命令；
- macOS 上的应用；
- Linux 上的菜单项、图标和 `magpie://` 链接处理；
- Linux 上会去启动一个已不存在的 magpie 的后台服务和开机启动。

保留：

- `~/.config/magpie` 里的设置、provider 和用量；
- 已连接的 Agent 仍指向 magpie 的地址（直到你恢复它们）。

### 17.3 彻底删除

```sh
curl -fsSL https://usemagpie.ai/install.sh | sh -s -- --uninstall --purge
```

`--purge` 会先运行 `magpie uninstall --yes`：

- 断开每个 Agent；
- 恢复它原来的配置；
- 删除 magpie 的设置、provider 和用量。

用其他方式安装的：

```sh
magpie uninstall
```

或设置 → 卸载 magpie（做的是 `--purge` 那些事，程序本身留给你删除）。

### 17.4 手动清理

如果安装脚本已经不在：

```sh
rm -rf ~/.config/magpie
rm -rf ~/.cache/magpie
# macOS
rm -rf ~/Library/Logs/magpie
# Linux
rm -rf ~/.local/state/magpie
```

> 删除前先确认已经用 `magpie <agent> default` 恢复了 Agent 配置，否则 Agent 可能还指向一个不存在的网关。

---

## 18. 术语表

| 术语 | 含义 |
| :-- | :-- |
| **Agent** | 使用模型的工具，如 Claude Code、Codex、Gemini CLI |
| **Provider** | 模型来源：订阅、API Key 厂商、本地模型、自定义地址 |
| **Model** | 具体模型，写作 `provider/model` |
| **Gateway** | 本地网关 `127.0.0.1:3425`，所有请求的入口 |
| **Routing group** | 把多个模型/账号合成一个，写作 `group/<id>` |
| **Profile / 方案** | 所有 Agent 配置的命名快照 |
| **Subscription** | 你登录的订阅（Claude、ChatGPT、Copilot 等） |
| **API Key provider** | 用密钥接入的厂商 |
| **Relay / 中转站** | 兼容 OpenAI/Anthropic 的第三方转发服务 |
| **Gateway key** | 给其他电脑/客户端使用的网关密钥 |
| **Remote magpie** | 把另一台电脑上的 magpie 当作 provider |
| **Failover** | 一个账号/模型失败时自动换下一个 |
| **Session stickiness** | 对话保持在回答它的账号上 |
| **Masking / 脱敏** | 请求离开电脑前把敏感值换成占位符 |
| **OTLP** | OpenTelemetry 导出协议 |
| **MCP** | Model Context Protocol，给 Agent 提供工具 |

---

## 19. 官方链接汇总

| 内容 | 链接 |
| :-- | :-- |
| 官网 / 下载 | <https://usemagpie.ai/zh/> |
| 中文快速上手 | <https://usemagpie.ai/docs/zh/start> |
| 中文插件文档 | <https://usemagpie.ai/docs/zh/plugins> |
| 中文意图路由 | <https://usemagpie.ai/docs/zh/intent> |
| 中文 Docker | <https://usemagpie.ai/docs/zh/docker> |
| 中文远程 magpie | <https://usemagpie.ai/docs/zh/remote> |
| 中文本地脱敏 | <https://usemagpie.ai/docs/zh/masking> |
| 中文隐私 | <https://usemagpie.ai/docs/zh/privacy> |
| 中文 MCP 路由 | <https://usemagpie.ai/docs/zh/mcp-router> |
| 中文导入链接 | <https://usemagpie.ai/docs/zh/import> |
| 完整参考（英文） | <https://github.com/yetone/magpie/blob/main/docs/reference.md> |
| 插件社区 | <https://github.com/magpie-community/plugins> |
| Discord | <https://discord.gg/vGSnD3ZKQF> |
| 发布记录 | <https://github.com/yetone/magpie-releases/releases> |

**许可：** MIT。magpie 与 Apache Magpie 及 Apache 软件基金会没有隶属关系。

**反馈：** 官方仓库不接受 Pull Request；问题、想法和模型缺失请去 Discord。

---

## 附录 A：常用命令清单

```sh
# ===== 状态 =====
magpie ls
magpie providers
magpie models
magpie presets
magpie quota
magpie usage 7d
magpie settings
magpie log -f

# ===== Provider =====
magpie provider add deepseek sk-…
magpie provider add ollama
magpie provider add "My Relay" url=https://relay.example.com/v1 key=sk-… models=gpt-5.5
magpie provider test deepseek
magpie provider models deepseek
magpie provider refresh deepseek
magpie provider key deepseek sk-…
magpie provider rm deepseek

# ===== Agent =====
magpie claude deepseek/deepseek-v4-pro
magpie codex moonshot/kimi-k2.5
magpie opencode anthropic/claude-sonnet-5
magpie codex effort high
magpie claude haiku deepseek/deepseek-v4-flash
magpie claude haiku ""
magpie claude default
magpie codex default

# ===== 方案 =====
magpie save work
magpie use work
magpie profiles
magpie rm work

# ===== 路由组 =====
magpie groups
magpie group add "Opus anywhere" models=claude/claude-opus-5-5,copilot/claude-opus-5.5 routing=smart
magpie group set opus-anywhere models+=openrouter/anthropic/claude-opus-5.5
magpie group set opus-anywhere routing=weight weight=claude/claude-opus-5-5=3
magpie group rule add opus-anywhere use=deepseek/deepseek-v4-flash compact
magpie group rule opus-anywhere
magpie group rm opus-anywhere
magpie claude group/opus-anywhere

# ===== 账号 =====
magpie accounts add codex
magpie accounts switch claude next
magpie accounts discovery off

# ===== 运行 =====
magpie
magpie tray
magpie tui
magpie web
magpie serve
magpie daemon install
magpie daemon status
magpie daemon restart
magpie healthcheck

# ===== 工具 =====
magpie tools
magpie tool generate_image "一只喜鹊，水彩" -o hero.png
magpie tool web_search "go 1.27 release notes"
magpie tool analyze_image ./shot.png --prompt "what does the error say?"

# ===== 维护 =====
magpie sync
magpie backup
magpie restore magpie.magpie-backup
magpie update
magpie update channel stable
magpie update mirror https://mirror.example/
magpie uninstall
```

---

## 附录 B：三套最小可用配置

### B.1 Claude Code + DeepSeek API Key

```sh
# 1. 安装
curl -fsSL https://usemagpie.ai/install.sh | sh

# 2. 添加 provider
magpie provider add deepseek sk-你的key

# 3. 选模型
magpie claude deepseek/deepseek-v4-pro

# 4. 验证
magpie ls
magpie provider test deepseek

# 5. 新开一个终端
claude
```

### B.2 Codex + Kimi API Key

```sh
# 1. 查看 Kimi / Moonshot 预设的 id（不同版本可能是 kimi 或 moonshot）
magpie presets | grep -i -E 'kimi|moonshot'

# 2. 用上一步看到的 id 添加（下面的 <id> 要替换成真实 id）
magpie provider add <id> sk-你的key

# 3. 确认模型的完整写法
magpie models | grep -i kimi

# 4. 给 Codex 选模型（下面的 <provider>/<model> 用上一步看到的完整写法）
magpie codex <provider>/<model>
magpie codex effort high
magpie ls

# 重启 Codex 应用，再开新会话
codex
```

> 如果 `magpie presets` 给出的 id 是 `moonshot`，把上面的 `<id>` 换成 `moonshot`，模型写法也以 `magpie models` 的输出为准。

### B.3 OpenCode + 本地 Ollama

```sh
# 先在本地启动 Ollama，并拉一个模型
ollama pull qwen3-coder

magpie provider add ollama
magpie opencode ollama/qwen3-coder
magpie ls
```

> 本地模型的 provider id 和模型名以 `magpie models` 的输出为准。

---

## 附录 C：上线前检查清单

- [ ] 已备份 `~/.claude/settings.json`、`~/.codex/config.toml` 等配置
- [ ] magpie 已安装并能启动（`magpie ls` 有输出）
- [ ] 至少一个 provider 测试通过（`magpie provider test <id>`）
- [ ] 目标 Agent 显示正确的 `provider/model`
- [ ] 新开会话后能正常对话
- [ ] `magpie usage today` 能看到请求
- [ ] 网关页 / 路由页能看到这次请求
- [ ] 代理软件已放行 `127.0.0.1`（如使用 TUN / 增强模式）
- [ ] 如使用 Docker，已生成并配置网关密钥
- [ ] 如使用局域网共享，已给每台电脑分配密钥
- [ ] 如使用订阅，已确认厂商的使用条款和封禁风险
- [ ] 如需隐私保护，已开启脱敏并用"试一试"验证
- [ ] 已了解 `magpie <agent> default` 可以恢复原样
- [ ] 已了解日志位置（`magpie log`）

---

> **最后一句：** magpie 的核心只有三步——**加 provider → 给 Agent 选模型 → 开新会话**。剩下的路由组、方案、脱敏、远程共享，都是在你需要时再逐层加上的可选能力。遇到问题先跑 `magpie ls`、`magpie provider test`、`magpie log -n 200`，大多数问题都能定位。
