# AI 编程工具清单

> 收录各 AI 编程工具在 **Windows / macOS / Linux** 上的官方安装方式、依赖环境与默认安装路径。
> 命令与路径均取自官方文档或官方安装脚本原文；三级形态中该厂商**没有的留空**。

## 分类

**一级：类型**


| 一级        | 说明                    |
| --------- | --------------------- |
| 一、本地模型运行时 | 模型跑在你自己机器上，对外提供本地推理服务 |
| 二、编程工具    | 直接读写代码的 agent         |
| 三、编排与网关   | 自身不写代码，调度其他工具或接入聊天客户端 |


**二级：出品方**（标注「模型厂商 / 第三方」）　**三级：形态**

Agent CLI（含 TUI）· IDE 插件 · Desktop App · Web 应用 / 云端 Agent · 移动端 App · 服务 / 网关（自托管）· 容器镜像（Docker）· SDK / 库

---

## 一、本地模型运行时

### ggml-org

**官网**：[https://llama.app/](https://llama.app/)
**GitHub**：`ggml-org/llama.cpp`（引擎源码）· `ggml-org/Llama-Windows`（Windows 端源码，**无 LICENSE 文件**）· `ggml-org/Llama-macOS`（macOS 端源码）· `ggml-org/llama-install.sh`（安装脚本仓库，MIT）

**作用**：由 **llama.cpp 团队（ggml-org）与 Hugging Face** 联合出品的本地模型运行时（llama.cpp 引擎）。Windows 11 / macOS 是桌面托盘应用，Linux 仅命令行；对外提供本地 OpenAI / Anthropic 兼容 API。**不是 Meta 的 Llama 工具**。

#### Agent CLI（含 TUI）

Windows｜依赖：无需 Node；GPU 加速需 CUDA Toolkit 或 ROCm / HIP SDK，否则回退 Vulkan / CPU

```powershell
irm https://llama.app/install.ps1 | iex
```

Linux｜依赖：仅需 curl；zstd 可选（缺失时脚本自行下载）

```bash
curl -LsSf https://llama.app/install.sh | sh
```

**安装路径**：Windows 可执行 `%LOCALAPPDATA%\Microsoft\WindowsApps\llama.exe`（脚本**不改 PATH**，依赖该目录默认在 PATH 中）、暂存 `%LOCALAPPDATA%\llama-app`｜Linux 可执行 `~/.local/bin/llama`、暂存 `~/.llama-app`

#### IDE 插件

#### Desktop App

Windows 11｜依赖：Windows 11 + Windows App Installer，x64 / ARM64

```
https://github.com/ggml-org/Llama-Windows/releases/download/v0.11.0/LlamaApp-v0.11.0.msixbundle
```

macOS｜依赖：macOS 15+，**仅 Apple Silicon**（依据 Homebrew cask 的 depends\_on）

```bash
brew install --cask llama-app
```

或 `.dmg`：`https://github.com/ggml-org/Llama-macOS/releases/latest/download/Llama.dmg`

**安装路径**：macOS `/Applications/Llama.app`；配置 `~/.config/llama/models.user.ini`（应用只读该文件并合并进它生成的 `models.ini`）｜Windows 设置与日志 `%LOCALAPPDATA%\Llama`

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

桌面端提供本地 API `http://localhost:9931/v1`；`llama serve` 默认 `http://127.0.0.1:8080`（含内置 WebUI）。空闲 5 分钟自动卸载模型。

#### 容器镜像（Docker）

`ghcr.io/ggml-org/llama.cpp`（属 llama.cpp 项目）

#### SDK / 库

**其他说明**：模型统一存放 Hugging Face 缓存，与 llama.cpp 及其他工具共享（Windows `%USERPROFILE%\.cache\huggingface\hub`）。⚠️ 官网「Package managers」链接指向的是 **llama.cpp** 文档——`winget install llama.cpp`、`brew install llama.cpp` 装的是 llama.cpp 本体，**不是 llama.app**，两者是不同项目、不同仓库。

### Ollama（模型厂商）

**官网**：[https://ollama.com/](https://ollama.com/)
**GitHub**：`ollama/ollama`（主源码，Go / MIT）· `ollama/ollama-python`、`ollama/ollama-js`（官方 SDK）

**作用**：本地大模型运行时，常驻服务 API `http://localhost:11434`；本地 GGUF 模型免费，云模型（`:cloud` 后缀）需 `ollama signin`。

#### Agent CLI（含 TUI）

Windows｜依赖：Windows 10 22H2+；NVIDIA 驱动 551.61+，AMD 需 ROCm v7 / HIP7 或 Vulkan 驱动

```powershell
irm https://ollama.com/install.ps1 | iex
```

macOS / Linux｜依赖：Linux 需 curl / awk / grep / sed / tee / xargs，且需 root 或 sudo（会创建 ollama 用户）；**拒绝 WSL1**；macOS 需 curl + unzip

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

指定版本

```bash
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

**安装路径**：Windows 二进制 `%LOCALAPPDATA%\Programs\Ollama`（安装器加入用户 PATH）｜macOS 应用 `/Applications/Ollama.app` + 软链 `/usr/local/bin/ollama`｜Linux 二进制为 PATH 中首个命中的 `/usr/local/bin`、`/usr/bin`、`/bin` 下，库文件在 `$OLLAMA_INSTALL_DIR/lib/ollama`

#### IDE 插件

VS Code 扩展 `Ollama.ollama`（[https://marketplace.visualstudio.com/items?itemName=Ollama.ollama](https://marketplace.visualstudio.com/items?itemName=Ollama.ollama)），需 VS Code 1.127+

#### Desktop App

macOS｜依赖：macOS Sonoma 14+

```
https://ollama.com/download/Ollama.dmg
```

Windows

```
https://ollama.com/download/OllamaSetup.exe
```

（支持 `/DIR="d:\some\location"` 自定义目录；Linux 无官方桌面端）

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

服务 API `http://localhost:11434`。Linux systemd 单元 `/etc/systemd/system/ollama.service`（`ExecStart=<BINDIR>/ollama serve`、`User=ollama`），覆盖片段放 `/etc/systemd/system/ollama.service.d/override.conf`；服务账号家目录 `/usr/share/ollama`。

Linux 手动安装（依赖 curl / tar / zstd）：

```bash
curl -fsSL https://ollama.com/download/ollama-linux-amd64.tar.zst | sudo tar x -C /usr
curl -fsSL https://ollama.com/download/ollama-linux-arm64.tar.zst | sudo tar x -C /usr
curl -fsSL https://ollama.com/download/ollama-linux-amd64-rocm.tar.zst | sudo tar x -C /usr   # AMD GPU 另装
```

#### 容器镜像（Docker）

```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
docker run -d --device /dev/kfd --device /dev/dri -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm
```

NVIDIA 需 NVIDIA Container Toolkit，AMD 需宿主 ROCm 驱动；容器内模型卷 `ollama:/root/.ollama`。

#### SDK / 库

```bash
pip install ollama
npm i ollama
```

**其他说明**：模型与配置在 `%HOMEPATH%\.ollama`（Windows）、`~/.ollama`（macOS）；Linux 模型在 `/usr/share/ollama/.ollama/models`。迁移模型目录用 `OLLAMA_MODELS`。日志：Windows `%LOCALAPPDATA%\Ollama`、macOS `~/.ollama/logs`。

---

## 二、编程工具

### Anthropic（模型厂商）

**官网**：[https://claude.com/product/claude-code](https://claude.com/product/claude-code)　文档 [https://code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview)
**GitHub**：`anthropics/claude-code`（**非源码仓库**，仅文档 + 插件示例 + devcontainer + issue 跟踪；LICENSE 为 "© Anthropic PBC. All rights reserved." 专有许可）· `anthropics/claude-agent-sdk-typescript`、`anthropics/claude-agent-sdk-python`（SDK 源码）

**作用**：Anthropic 官方 agentic coding CLI（命令 `claude`）。需 Pro / Max / Team / Enterprise 或 Console 账号，**免费版 claude.ai 计划不含 Claude Code**。

#### Agent CLI（含 TUI）

Windows｜依赖：Windows 10 1809+ / Server 2019+；4 GB+ 内存；x64 或 ARM64；无需 Node；Git for Windows 可选但推荐（提供 Bash 工具）

```powershell
irm https://claude.ai/install.ps1 | iex
```

```powershell
# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

指定 stable 渠道 / 版本号：

```powershell
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) stable
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) 2.1.89
```

macOS｜依赖：macOS 13.0+；curl 或 wget；无需 Node

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Linux｜依赖：Ubuntu 20.04+ / Debian 10+ / Alpine 3.19+

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Alpine 前置｜依赖：apk

```sh
apk add bash curl libgcc libstdc++ ripgrep
```

并设置 `USE_BUILTIN_RIPGREP=0`。

npm（跨平台）｜依赖：Node.js 22+；**禁止 `sudo npm install -g`**；包内为原生二进制，低于 22 只打印 EBADENGINE 警告

```bash
npm install -g @anthropic-ai/claude-code
```

Debian / Ubuntu 官方 apt 仓库｜依赖：sudo、curl、gnupg

```bash
sudo apt install curl gnupg
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://downloads.claude.ai/keys/claude-code.asc -o /etc/apt/keyrings/claude-code.asc
echo "deb [signed-by=/etc/apt/keyrings/claude-code.asc] https://downloads.claude.ai/claude-code/apt/stable stable main" | sudo tee /etc/apt/sources.list.d/claude-code.list
sudo apt update && sudo apt install claude-code
```

Fedora / RHEL 官方 dnf 仓库

```bash
sudo tee /etc/yum.repos.d/claude-code.repo <<'EOF'
[claude-code]
name=Claude Code
baseurl=https://downloads.claude.ai/claude-code/rpm/stable
enabled=1
gpgcheck=1
gpgkey=https://downloads.claude.ai/keys/claude-code.asc
EOF
sudo dnf install claude-code
```

Alpine 官方 apk 仓库｜依赖：wget

```sh
wget -O /etc/apk/keys/claude-code.rsa.pub https://downloads.claude.ai/keys/claude-code.rsa.pub
echo "https://downloads.claude.ai/claude-code/apk/stable" >> /etc/apk/repositories
apk add claude-code
```

**安装路径**：Windows 可执行 `%USERPROFILE%\.local\bin\claude.exe`、版本 `%USERPROFILE%\.local\share\claude`｜macOS / Linux `~/.local/bin/claude`（软链指向 `~/.local/share/claude/versions/`）｜配置 `~/.claude` 与 `~/.claude.json`，项目级 `.claude`、`.mcp.json`。原生安装**后台自动更新**；WinGet / Homebrew 需手动升级。

#### IDE 插件

VS Code 扩展 `anthropic.claude-code`（[https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code)），需 VS Code 1.98+
JetBrains 插件 [https://plugins.jetbrains.com/plugin/27310-claude-code-beta-](https://plugins.jetbrains.com/plugin/27310-claude-code-beta-)，**依赖已安装的 `claude` CLI**

#### Desktop App

macOS / Windows / Linux（beta）｜依赖：付费订阅

```
https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect
```

官方描述为 "A standalone app for running Claude Code"（Claude 应用的 Code 标签）

#### Web 应用 / 云端 Agent

[https://claude.ai/code](https://claude.ai/code)

#### 移动端 App

Claude 的 iOS / Android App 内含 Code 标签。官方原文："Claude Code doesn't have a separate mobile app."

#### 服务 / 网关（自托管）

自托管环境（K8s / Compose，Team / Enterprise beta）

#### 容器镜像（Docker）

`ghcr.io/anthropics/devcontainer-features/claude-code:1.0`（Dev Container Feature，无独立官方镜像）

#### SDK / 库

`@anthropic-ai/claude-agent-sdk`（TypeScript）、`claude-agent-sdk`（Python）

**其他说明**：另有 WinGet `winget install Anthropic.ClaudeCode`、Homebrew `brew install --cask claude-code`（latest 渠道为 `claude-code@latest`）。原生安装脚本**拒绝以 sudo / root 运行**。

### OpenAI（模型厂商）

**官网**：[https://chatgpt.com/codex](https://chatgpt.com/codex)　文档 [https://learn.chatgpt.com/docs](https://learn.chatgpt.com/docs)
**GitHub**：`openai/codex`（**主源码**，Rust / JS，Apache-2.0）· `openai/codex-universal`（基础镜像仓库，不含 CLI）

**作用**：OpenAI 官方编码 agent。用 ChatGPT 订阅登录或 API key 计费。

#### Agent CLI（含 TUI）

Windows｜依赖：Windows PowerShell（脚本需绕过执行策略）；原生 Windows 是官方一等路径

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

强制走 GitHub Releases：

```powershell
$env:CODEX_INSTALLER_USE_RELEASES_OPENAI_COM='false'; irm https://chatgpt.com/codex/install.ps1 | iex
```

macOS / Linux｜依赖：macOS 12+ / Ubuntu 20.04+ / Debian 10+；mktemp、tar；curl 或 wget；校验需 sha256sum / shasum / openssl 之一

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

npm（跨平台）｜依赖：Node.js >= 16（该包 engines 原文）

```bash
npm install -g @openai/codex
```

**安装路径**：Windows 可见 bin `%LOCALAPPDATA%\Programs\OpenAI\Codex\bin`（junction，`CODEX_INSTALL_DIR` 可改），实体在 `%USERPROFILE%\.codex\packages\standalone\releases\<版本>-<target>\`，`current` 为 junction｜macOS / Linux 可执行 `~/.local/bin/codex`，版本包 `~/.codex/packages/standalone/releases/<版本>-<target>`｜配置 `~/.codex/config.toml`（`CODEX_HOME` 可改），系统级 `/etc/codex/config.toml`（Unix）

#### IDE 插件

VS Code 扩展 `openai.chatgpt`（[https://marketplace.visualstudio.com/items?itemName=openai.chatgpt](https://marketplace.visualstudio.com/items?itemName=openai.chatgpt)），支持 VS Code / Cursor / Windsurf

#### Desktop App

Windows｜依赖：Microsoft Store

```powershell
winget install --id 9PLM9XGG6VKS -s msstore
```

macOS 用 `Codex.dmg`｜Linux 见官方 `/codex/linux`｜三者均可用 CLI 入口 `codex app`

#### Web 应用 / 云端 Agent

Codex Cloud（chatgpt.com/codex）

#### 移动端 App

Codex Remote —— 用 ChatGPT 手机 App 配对电脑

#### 服务 / 网关（自托管）

`codex app-server`（JSON-RPC）

#### 容器镜像（Docker）

#### SDK / 库

`@openai/codex-sdk`（TypeScript）、`openai-codex`（Python）

**其他说明**：另有 Homebrew `brew install --cask codex`。Git 2.23+ 可选（PR 辅助）。⚠️ **WSL1 自 Codex 0.115 起不再支持**；WSL2 是按需可选项（官方 WSL 页原文："Choose WSL2 when you need Linux-native tooling..."），不再是推荐路径。

### Google（模型厂商）

**官网**：[https://antigravity.google/](https://antigravity.google/)　文档 [https://antigravity.google/docs/cli/overview](https://antigravity.google/docs/cli/overview)
**GitHub**：`google-antigravity/antigravity-cli`（**非源码仓库**，仅 issue 模板 + examples + CHANGELOG；**CLI 自身源码仓库未确认**）· `google-antigravity/antigravity-sdk-python`（SDK 源码）

**作用**：Google agentic 开发平台的终端 CLI，二进制名 `agy`；支持 shell 命令与后台 subagent。

#### Agent CLI（含 TUI）

Windows PowerShell｜依赖：64 位 Windows；`Get-FileHash` 不可用时回退 `certutil` 校验 SHA-512

```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

Windows CMD

```powershell
curl -fsSL https://antigravity.google/cli/install.cmd -o install.cmd && install.cmd && del install.cmd
```

macOS / Linux｜依赖：curl 或 wget；tar、sed；校验用 shasum / sha512sum；**无需 Node**

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

**安装路径**：Windows `%LOCALAPPDATA%\agy\bin\agy.exe`（`-d` / `--dir` 可改，脚本**不改 PATH**）｜macOS / Linux `~/.local/bin/agy`、暂存 `~/.cache/antigravity/staging`｜配置 `~/.gemini/antigravity-cli/settings.json`、`keybindings.json`；凭据存于系统 keyring（Keychain / Secret Service / Windows Credential Manager）

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`google-antigravity/antigravity-sdk-python`

**其他说明**：脚本可选参数 `--skip-aliases`（不改动已有 `agy` / `antigravity` 别名）、`--skip-path`（不改 shell profile）。个人计划 $0（有限周限额），企业经 Google Cloud 起价 $30 USD / 席位 / 月。⚠️ 本工具**无任何许可证声明**（仓库 LICENSE / NOTICE 均 404，官方 terms 页检索无结果）。CLI 页面本身未标注最低系统版本。

### SpaceXAI（模型厂商）

**官网**：[https://x.ai/cli](https://x.ai/cli)　文档 [https://docs.x.ai/build/overview](https://docs.x.ai/build/overview)
**GitHub**：`xai-org/grok-build`（**源码仓库**，Rust / Apache-2.0）

**作用**：Grok Build —— 全屏 TUI 编码 agent（命令 `grok`）；支持交互、headless（CI）与编辑器 ACP 嵌入。

#### Agent CLI（含 TUI）

Windows｜依赖：Windows PowerShell（强制 TLS 1.2）；仅对 MinGit 压缩包做 SHA-256 校验，**主程序不做哈希校验**

```powershell
irm https://x.ai/cli/install.ps1 | iex
```

macOS / Linux / Git Bash｜依赖：curl 或 wget（至少其一）；zstd 可选（缺失退 gzip，再退未压缩）；**不做校验和验证**

```bash
curl -fsSL https://x.ai/cli/install.sh | bash
```

**安装路径**：Windows `%USERPROFILE%\.grok\bin`（`GROK_BIN_DIR` 可改，写入用户级 PATH）；MinGit 解压到 `%LOCALAPPDATA%\grok\git\<版本>\`｜macOS / Linux `~/.grok/bin`（落为 `grok-<platform>` 并软链 `grok`、`agent`；bin 目录不在 PATH 时会软链到 `~/.local/bin` 或 `/usr/local/bin`）｜配置 `~/.grok/config.toml`；凭据 `~/.grok/auth.json`；补全 `~/.grok/completions/`

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：另有 WinGet 包 `xAI.GrokBuild`、`winget upgrade --id xAI.GrokBuild -e`。指定版本先设 `$env:GROK_VERSION`，渠道变量 `GROK_CHANNEL`（默认 `stable`）。源码构建需 Rust（rustup）+ DotSlash + protoc，且**仅支持 macOS / Linux 宿主**，Windows 构建为 best-effort 且未测试。官方渠道**未提供 Homebrew / npm / Docker**。

### Moonshot AI（模型厂商）

**官网**：[https://www.kimi.com/code](https://www.kimi.com/code)
**GitHub**：`MoonshotAI/kimi-code`（现行主源码，TS / MIT）· `MoonshotAI/kimi-cli`（**已归档只读**）· `MoonshotAI/kimi-agent-sdk`、`MoonshotAI/kimi-code-zed-extension`、`MoonshotAI/kimi-agent-rs`

**作用**：Kimi Code CLI（命令 `kimi`）；支持 MCP 对话式配置、子 Agent 并行、ACP 集成；订阅制。

#### Agent CLI（含 TUI）

Windows｜依赖：PowerShell 5.1+；脚本无需 Node；**运行时需 Git for Windows**（用其 Git Bash 作 Shell）

```powershell
irm https://code.kimi.com/kimi-code/install.ps1 | iex
```

macOS / Linux｜依赖：curl 或 wget；**校验必需** shasum 或 sha256sum；**仅提供 glibc 构建，检测到 musl / Alpine 直接中止**并要求改用 npm

```bash
curl -fsSL https://code.kimi.com/kimi-code/install.sh | bash
```

npm / pnpm（跨平台）｜依赖：Node.js 22.19.0+

```bash
npm install -g @moonshot-ai/kimi-code
pnpm add -g @moonshot-ai/kimi-code
```

**安装路径**：Windows `%USERPROFILE%\.kimi-code\bin\kimi.exe`（`KIMI_INSTALL_DIR` 可改）｜macOS / Linux `~/.kimi-code/bin/kimi`｜配置与数据 `~/.kimi-code/`（`KIMI_CODE_HOME` 可改），含 `config.toml`、会话、日志；脚本写 `region` 标记文件。脚本会向 `~/.zshrc` / `~/.bashrc` / fish 配置追加 PATH（`KIMI_NO_MODIFY_PATH` 可跳过）。

#### IDE 插件

VS Code 扩展 `moonshot-ai.kimi-code`｜Zed 官方扩展（`MoonshotAI/kimi-code-zed-extension`）｜JetBrains 走 ACP 接入（无市场插件）

#### Desktop App

`KimiCode-mac-arm64.dmg` / `-mac-x64.dmg` / `-win-x64.exe`（`code.kimi.com/kimi-code/desktop/download/…`）｜依赖：Kimi 账号；落盘官方未说明

#### Web 应用 / 云端 Agent

`kimi web` → `127.0.0.1:58627`；Remote Control 网页 `code-rc.kimi.com`（付费）

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`@moonshot-ai/kimi-agent-sdk`（TS）、`kimi-agent-sdk`（PyPI）

**其他说明**：Git Bash 在非标准路径时需设 `KIMI_SHELL_PATH`。升级 `kimi upgrade`，卸载 npm 安装用 `npm uninstall -g @moonshot-ai/kimi-code`。

### Qwen（模型厂商）

**官网**：[https://qwenlm.github.io/qwen-code-docs/](https://qwenlm.github.io/qwen-code-docs/)
**GitHub**：`QwenLM/qwen-code`（**主源码 monorepo**，TS / Apache-2.0，桌面端与 SDK 同仓发布）· `QwenLM/qwen-code-docs`（文档站源码）· `QwenLM/qwen-code-action`

**作用**：阿里巴巴 Qwen 团队的开源终端编码 agent（命令 `qwen`），自带 Auto-Memory / SubAgents / Agent Teams / MCP / Plan Mode / LSP / Sandbox；可接 OpenAI、Anthropic、Gemini、Qwen 及本地 Ollama / vLLM。

#### Agent CLI（含 TUI）

Windows｜依赖：**standalone 包自带 Node 运行时**，无需预装；脚本只把 bin 前置到当前会话 PATH

```powershell
irm https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen-standalone.ps1 | iex
```

macOS / Linux｜依赖：bash（非 bash 会自行 re-exec）、curl 或 wget、tar、sha256sum 或 shasum；**自带 Node**；仅覆盖 darwin / linux

```bash
curl -fsSL https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen-standalone.sh | bash
```

npm（跨平台）｜依赖：Node.js 22+

```bash
npm install -g @qwen-code/qwen-code@latest
```

Homebrew（macOS / Linux，社区维护）

```bash
brew install qwen-code
```

**安装路径**：Windows `%LOCALAPPDATA%\qwen-code\bin\qwen.cmd`（`QWEN_INSTALL_ROOT` 可改）｜macOS / Linux 可执行 `~/.local/bin/qwen`、程序本体 `~/.local/lib/qwen-code`｜配置 `~/.qwen/`（含 `source.json`）、项目级 `<project>/.qwen/`

#### IDE 插件

VS Code 扩展 `qwenlm.qwen-code-vscode-ide-companion`（VS Code ≥ 1.96）｜Zed 集成｜JetBrains 走 ACP

#### Desktop App

[https://github.com/QwenLM/qwen-code/releases/tag/desktop-latest](https://github.com/QwenLM/qwen-code/releases/tag/desktop-latest)（desktop-v0.0.5；dmg / exe / AppImage）｜落盘官方未说明

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

`qwen serve` → `127.0.0.1:4170` 内置 Web Shell；`--local-control` 可手机扫码（Stage 1 experimental）

#### 容器镜像（Docker）

`ghcr.io/qwenlm/qwen-code`

#### SDK / 库

`@qwen-code/sdk`（TS）、`qwen-code-sdk`（Python，alpha）、`com.alibaba:qwencode-sdk:0.1.0-alpha`（Java）

**其他说明**：脚本向 `~/.zshrc` / `~/.bashrc` / fish 配置追加带标记的 PATH 块（`--no-modify-path` 可跳过）。npm 路径**不会**代为安装 Node，也不会改 npm config；用脚本时默认 registry 为 `registry.npmmirror.com`。⚠️ **Qwen OAuth 已于 2026-04-15 停止服务**。

### Xiaomi MiMo（模型厂商）

**官网**：[https://mimo.xiaomi.com/mimocode](https://mimo.xiaomi.com/mimocode)
**GitHub**：`XiaomiMiMo/MiMo-Code`（**源码仓库**，TS / MIT，OpenCode fork，SDK 同仓）· `XiaomiMiMo/MiMo-Skills` · `XiaomiMiMo/awesome-mimo-agent`（非源码，策展文档）

**作用**：MiMo Code（命令 `mimo`）——终端 AI 编程助手；持久记忆（SQLite FTS5）、多 Agent、宣称无限上下文。许可证 MIT **附带 `USE_RESTRICTIONS.md` 使用限制**。

#### Agent CLI（含 TUI）

Windows｜依赖：PowerShell 5+；脚本无 Node 依赖

```powershell
powershell -ep Bypass -c "irm https://mimo.xiaomi.com/install.ps1 | iex"
```

macOS / Linux｜依赖：Linux 需 tar、其他平台需 unzip；curl；**x64 缺 AVX2 时自动回退 `-baseline` 变体**；探测 musl 以选构建

```bash
curl -fsSL https://mimo.xiaomi.com/install | bash
```

npm（跨平台）｜依赖：npm（包内 `engines` / `os` / `cpu` **均未声明**；postinstall 为 `bun ./postinstall.mjs || node ./postinstall.mjs`，优先 bun、缺失回退 node）

```bash
npm install -g @mimo-ai/cli
```

**安装路径**：Windows `%USERPROFILE%\.mimocode\bin\mimo.exe`（`MIMOCODE_INSTALL_DIR` 可改；默认前插用户 PATH，`-NoModifyPath` 可跳过）｜macOS / Linux `~/.mimocode/bin/mimo`｜配置 `~/.config/mimocode/mimocode.jsonc`（Windows 落在 `%LOCALAPPDATA%\mimocode\`）、项目级 `.mimocode/mimocode.jsonc`｜数据 macOS `~/Library/Application Support/mimocode/`、Linux `~/.local/share/mimocode/`｜凭据 `~/.local/share/mimocode/auth.json`（`MIMOCODE_HOME` 可重定位全部路径）

#### IDE 插件

#### Desktop App

暂列于 [https://mimo.xiaomimimo.com/desktop/](https://mimo.xiaomimimo.com/desktop/)（win-x64 setup.exe、mac-arm64 dmg）。⚠️ 口径有争议：官方 README 称 "powered by MiMo Code as its core engine"，桌面页仅称「Xiaomi MiMo 桌面客户端」，**是否属于 MiMo Code 自身的桌面端未确认**

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

`mimo serve --port 4096` + `mimo attach http://127.0.0.1:4096`；`mimo run` 为无头模式

#### 容器镜像（Docker）

#### SDK / 库

`@mimo-ai/sdk`

**其他说明**：macOS 内置 Terminal.app **不支持**，需 iTerm2 或 VS Code 终端；WSL 剪贴板可能需 `xsel`。首启可选 Xiaomi MiMo Platform（OAuth）、Codex、从 Claude Code 导入或自定义 provider。

### Cognition（模型厂商）

**官网**：[https://cli.devin.ai](https://cli.devin.ai)（301 → docs.devin.ai/cli，无独立营销站）
**GitHub**：`CognitionAI/devin-cli`（**非源码仓库**，仅标题式 README + workflows/scripts，2 个提交、无 LICENSE）；**产品无公开源码仓库**

**作用**：Devin CLI（命令 `devin`）——本地命令行编码 agent，"Devin for Terminal"。**与云 VM 里的 Devin 是两个不同产品**。

#### Agent CLI（含 TUI）

Windows｜依赖：官方未列运行时依赖；**必须用 PowerShell**（Git Bash / CMD 会失败）；**不要以管理员运行**；脚本自动校验 SHA-256

```powershell
irm https://static.devin.ai/cli/setup.ps1 | iex
```

Windows 安装器直链（x86\_64 / ARM64）

```
https://static.devin.ai/cli/devin-updater-x86_64-pc-windows.exe
https://static.devin.ai/cli/devin-updater-aarch64-pc-windows.exe
```

macOS / Linux｜依赖：curl、tar、sha256sum（或 shasum）；无需 Node / Python

```bash
curl -fsSL https://cli.devin.ai/install.sh | bash
```

macOS Homebrew（社区维护）

```bash
brew install --cask devin-cli
```

**安装路径**：Windows `%LOCALAPPDATA%\devin\cli\bin\devin.exe`，版本体 `%LOCALAPPDATA%\devin\cli\_versions\<版本>\`｜macOS / Linux `~/.local/bin/devin`（软链 → `${XDG_DATA_HOME:-~/.local/share}/devin/cli/_versions/<版本>/bin/devin`，`current` 为当前版本链接；man page 在 `${XDG_DATA_HOME}/man/man1`）｜配置 Windows `%APPDATA%\devin\config.json` 与 `mcp_config.json`、macOS / Linux `~/.config/devin/config.json` 与 `AGENTS.md`；项目级 `.devin/config.json`｜日志 Windows `%APPDATA%\devin\cli\logs\`

#### IDE 插件

JetBrains / Zed / Xcode 的 ACP 接入

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：执行策略报错时用 `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`。企业客户可随 Devin Desktop 捆绑安装（限 Legacy Windsurf Enterprise / Devin Enterprise，需管理员先在团队设置开启 "Install Devin CLI in Devin Desktop"）。收费 Free $0 / Pro $20 / Max $200（每月）。

### Nous Research（模型厂商）

**官网**：[https://hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com)
**GitHub**：`NousResearch/hermes-agent`（**完整源码仓库**，MIT，含 `docker/`、`Dockerfile`、`flake.nix`）

**作用**：Hermes Agent —— 自改进 AI agent；TUI + 单网关接入 Telegram / Discord / Slack / WhatsApp / Signal。

#### Agent CLI（含 TUI）

Windows｜依赖：PowerShell 7+；安装器**全托管** Python 3.14、Node.js、npm、ripgrep、FFmpeg，并固定下载自己校验过的 `uv`；缺 Git 时自动下载便携 Git

```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

macOS / Linux / WSL2｜依赖：Git、curl、tar、SHA-256 工具；**Python 3.14**（`pyproject.toml` 区间 `>=3.11,<3.15`）；glibc Linux 下托管 Node 需 `libatomic.so.1`

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**安装路径**：Windows 代码 `%LOCALAPPDATA%\hermes\hermes-agent\`、入口与运行时 `%LOCALAPPDATA%\hermes\bin\`、数据 `%LOCALAPPDATA%\hermes\`｜macOS / Linux 代码 `~/.hermes/hermes-agent/`、入口 `~/.local/bin/hermes`、数据 `~/.hermes/`（`HERMES_HOME` 可选，`--dir` / `-HermesHome` / `-InstallDir` 可覆盖）｜配置 `~/.hermes/config.yaml`；凭据 `~/.hermes/auth.json`；日志 `logs/install.log`

#### IDE 插件

#### Desktop App

macOS｜依赖：macOS 12+，**仅 Apple Silicon**（Intel 的 DMG 会报 "not supported on this Mac"）

```
Hermes-Setup.dmg
```

Windows `Hermes-Setup.exe` / `.appinstaller`｜Linux 无安装包（用 `hermes desktop`）
**安装路径**：macOS 拖入 `/Applications`；数据仍在 `~/.hermes/`

#### Web 应用 / 云端 Agent

Web Dashboard `hermes dashboard` → `127.0.0.1:9119`；云托管 [https://portal.nousresearch.com/cloud](https://portal.nousresearch.com/cloud)

#### 移动端 App

#### 服务 / 网关（自托管）

`hermes gateway install`（systemd / launchd / 计划任务）；macOS plist `~/Library/LaunchAgents/ai.hermes.gateway.plist`，日志 `~/.hermes/logs/gateway.log`；API Server `127.0.0.1:8642/v1`

#### 容器镜像（Docker）

`nousresearch/hermes-agent`｜落盘：代码 `/opt/hermes/`，数据挂载 `/opt/data/`；⚠️ 官方注明 "Docker installs do not support `hermes update`"

#### SDK / 库

**其他说明**：⚠️ **本清单中唯一硬性要求 Python 的工具**，且明确**不支持** `uv tool install` / `pip install` / `brew install`。可选组件用 `-SkipBrowser` / `-SkipComputerUse` 跳过（会被记住）。Termux（Android，仅 aarch64）渠道已官方签名，但官方明确警告该包「does not work right now」。Windows Defender 可能误报 `bin\uv.exe`。

### Anysphere（第三方）

**官网**：[https://cursor.com/cli](https://cursor.com/cli)　文档 [https://cursor.com/docs/cli/overview](https://cursor.com/docs/cli/overview)
**GitHub**：**无公开源码仓库**（`cursor/cursor` 为元 / 反馈仓库，仅 issue 模板与 README）

**作用**：Cursor CLI —— 命令为 **`agent`**；支持 Agent / Plan / Ask 模式与 print 非交互模式（`agent -p`）。

#### Agent CLI（含 TUI）

Windows｜依赖：Windows PowerShell（脚本用 `Get-WmiObject` 判 ARM64、`Expand-Archive`、`Invoke-WebRequest`）

```powershell
irm 'https://cursor.com/install?win32=true' | iex
```

macOS / Linux / WSL｜依赖：bash、curl、tar、uname、ln、mv；仅支持 x64 / arm64；无需 sudo

```bash
curl https://cursor.com/install -fsS | bash
```

**安装路径**：Windows `%LOCALAPPDATA%\cursor-agent\`，版本在 `versions\<版本号>\`，主命令 `agent.exe`（由 `cursor-agent.exe` 复制出别名）；脚本**每次安装会先递归删除该目录**，并写入用户级 PATH｜macOS / Linux `~/.local/share/cursor-agent/versions/<版本号>/`，软链 `~/.local/bin/agent` 与 `~/.local/bin/cursor-agent`｜配置 `~/.cursor/cli-config.json`（`CURSOR_CONFIG_DIR` 可改；Linux/BSD 支持 `XDG_CONFIG_HOME`），项目级 `<project>/.cursor/cli.json`（仅可配 permissions）

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：验证 `agent --version`，更新 `agent update`（默认自动更新）。需把 `~/.local/bin` 加入 PATH。收费 Hobby 免费 / Individual $20 / Teams $40（每人每月）。

### TRAE（第三方）

**官网**：[https://trae.cn/trae-cli](https://trae.cn/trae-cli)　文档 [https://docs.trae.cn/cli\_get-started-with-trae-code-cli-2](https://docs.trae.cn/cli_get-started-with-trae-code-cli-2)
**GitHub**：**未找到官方源码仓库**（`bytedance/trae-agent` 是另一个研究项目，不计）

**作用**：字节跳动的 Trae CLI（命令 `traecli`，实际二进制 `traex`）—— TUI 交互式编码；仅调用内置模型消耗积分，自定义模型不消耗。

#### Agent CLI（含 TUI）

Windows｜依赖：**建议优先 WSL2**；原生版首次启动需按提示完成沙箱初始化

```powershell
irm https://trae.cn/trae-cli/install_v2.ps1 | iex
```

macOS / Linux｜依赖：curl、gzip、tar、sha256sum 或 shasum；`python3` 可选（缺失用 perl / sed 兜底）；脚本按需自动装 ripgrep 与 bubblewrap（Linux）

```bash
sh -c "$(curl -fsSL https://trae.cn/trae-cli/install_v2.sh)"
```

**安装路径**：Windows `%LOCALAPPDATA%\Programs\TraeCLI\bin`（internal 通道为 `Programs\TraeX\bin`）、状态 `%LOCALAPPDATA%\TraeCLI`｜macOS / Linux 可执行 `${TRAECLI_INSTALL_DIR:-${XDG_BIN_HOME:-~/.local/bin}}`（主程序 `traex`，另有 `traecli` / `trae-cli` / `trae-agent` / `ta` 链接）、状态 `${XDG_DATA_HOME:-~/.local/share}/traecli`｜全局配置 `trae_cli.yaml`：Windows `%APPDATA%\trae_cli\`、macOS `~/Library/Application Support/trae_cli/`、Linux `$XDG_CONFIG_HOME/trae_cli/` 或 `~/.config/trae_cli/`；用户级 `~/.trae/traecli.toml`

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：⚠️ **旧版 `install.ps1` 仍在线但已非官方推荐**（旧版装到 `%LOCALAPPDATA%\trae-cli\bin\trae-cli.exe`，路径与可执行文件名都不同）。⚠️ **国际版 docs.trae.ai 的 CLI 页面已 302 重定向下线**（跳向 TRAE SOLO 公告页），目前只有中国版 docs.trae.cn 维护 CLI 文档。登录：`traecli login status`，或用 `TRAECLI_PERSONAL_ACCESS_TOKEN` + `traecli login --with-trae-pat`。

### Amazon Web Services（第三方）

**官网**：[https://kiro.dev](https://kiro.dev)｜CLI 落地页 [https://cli.kiro.dev](https://cli.kiro.dev)
**GitHub**：`kirodotdev/Kiro`（**仅 issue / 反馈跟踪，明确不含源码** —— README 原文 "The Kiro product source code is not hosted here."）

**作用**：Kiro CLI（命令 `kiro-cli`）—— Rust 原生二进制，非 Node 应用。

#### Agent CLI（含 TUI）

Windows｜依赖：**Windows 11 + Windows Terminal 或 PowerShell**（CMD 不适用）；仅 x86\_64

```powershell
irm 'https://cli.kiro.dev/install.ps1' | iex
```

（脚本下载 `kiro-cli-x86_64-pc-windows-msvc.msi`，SHA256 校验后 `msiexec /i ... /quiet /norestart`）

macOS｜依赖：curl 或 wget、shasum、`hdiutil` / `ditto`；临时目录需 ≥ 2 GiB

```bash
curl -fsSL https://cli.kiro.dev/install | bash
```

Linux｜依赖：curl 或 wget、unzip、sha256sum；**glibc 2.34+**，低于门槛自动改用 musl 变体

```bash
curl -fsSL https://cli.kiro.dev/install | bash
```

**安装路径**：Windows `C:\Program Files\Kiro-Cli\`｜macOS 为 app bundle，用 `ditto` 复制到 `/Applications`（安装后 `open -g -a ... --no-dashboard`）｜Linux `~/.local/bin`（`kiro-cli`、`kiro-cli-chat`）｜配置全局 `~/.kiro/`（Windows 即 `%USERPROFILE%\.kiro`；`KIRO_HOME` 可整体重定向），含 `settings/cli.json`、`settings/mcp.json`、`agents/`、`steering/`、`skills/`、`hooks/`｜日志：Windows `%TEMP%\kiro-log\logs\kiro-chat.log`、macOS `$TMPDIR/kiro-log/kiro-chat.log`、Linux `$XDG_RUNTIME_DIR/kiro-log/kiro-chat.log`

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：无官方 Homebrew 方式；脚本不捆绑运行时，也不需要 Node / Python。登录 `kiro-cli login`，排障 `kiro-cli doctor`。收费 Free $0（50 credits）/ Pro $20 / Pro+ $40 / Pro Max $100 / Power $200（每用户每月）。Kiro IDE / Web / Mobile / Crew 属其他产品，与本 CLI 共用 `.kiro` 配置。

### Cline Bot（第三方）

**官网**：[https://cline.bot](https://cline.bot)　文档 [https://docs.cline.bot](https://docs.cline.bot)
**GitHub**：`cline/cline`（**完整源码 monorepo**，Apache-2.0；`apps/cli`、`sdk/` 在顶层）· `cline/kanban`（research preview）· `cline/homebrew-cline`（官方 tap）

**作用**：Cline —— 命令行 TUI + headless JSON 模式（CI/CD）。软件免费，仅按推理用量付费。

#### Agent CLI（含 TUI）

全平台｜依赖：**Node.js 20+（推荐 22）**

```bash
npm install -g cline
```

**安装路径**：可执行由 **npm 全局前缀**决定（Windows `%APPDATA%\npm`，全局包在 `%APPDATA%\npm\node_modules`，可执行 shim 直接在 `%APPDATA%\npm` 且需在 PATH 中）｜数据与配置 `~/.cline`（`CLINE_DATA_DIR` 或 `--data-dir` 可改；`--config` 默认 `~/.cline/data/settings`；hooks 默认 `~/.cline/hooks`）

#### IDE 插件

VS Code 系 `saoudrizwan.claude-dev`（[https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev)；支持 VS Code / Cursor / Windsurf / VSCodium，Windsurf 与 VSCodium 走 Open VSX）
JetBrains [https://plugins.jetbrains.com/plugin/28247-cline](https://plugins.jetbrains.com/plugin/28247-cline)

#### Desktop App

[https://cline.bot/desktop](https://cline.bot/desktop)（beta v0.0.43）｜依赖：macOS 与 Windows（Windows 当前 beta，Linux 未提及）；落盘官方未说明

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`@cline/sdk`（Node ≥ 22）；另有 Kanban（preview）`npx kanban`，依赖 Node.js 18+

**其他说明**：官方**没有** curl 安装脚本、Homebrew 或系统包管理器方式，三平台都只有 npm。

### Kilo（第三方）

**官网**：[https://kilo.ai](https://kilo.ai)　文档 [https://kilo.ai/docs](https://kilo.ai/docs)
**GitHub**：`Kilo-Org/kilocode`（**完整源码仓库**，TS / MIT；`packages/` 含 opencode 运行时、tui、kilo-vscode、kilo-jetbrains、sdk、server，**无单独 CLI 仓库**）· `Kilo-Org/kilo.nvim`（官方 Neovim 插件源码）· `Kilo-Org/homebrew-tap`

**作用**：Kilo CLI（已被 Anaconda 收购）—— OpenCode 的 fork；500+ 模型、零加价 BYOK、并行多 Agent、`/sandbox` 沙箱。

#### Agent CLI（含 TUI）

Windows｜依赖：Node 最低版本**上游未声明**（npm 包无 `engines` 字段）

```bash
npm install -g @kilocode/cli
```

macOS / Linux｜依赖：curl；Linux 需 tar、macOS 需 unzip；**不需要 Node**

```bash
curl -fsSL https://kilo.ai/cli/install | bash
```

其他方式

```bash
pnpm add -g @kilocode/cli
bun add -g @kilocode/cli
brew install Kilo-Org/tap/kilo
```

**安装路径**：Windows 由 npm 全局前缀决定｜macOS / Linux `~/.kilo/bin/kilo`｜全局配置 `~/.config/kilo/kilo.json[c]`（旧名 `opencode.json[c]`）、`tui.jsonc`——⚠️ 官方注明 **"Windows config dir may vary"**；项目级 `./kilo.json[c]` 或 `./.kilo/`（兼容 `./.kilocode/`）

#### IDE 插件

VS Code `kilocode.kilo-code`（Marketplace + Open VSX）｜JetBrains [https://plugins.jetbrains.com/plugin/28350-kilo-code](https://plugins.jetbrains.com/plugin/28350-kilo-code)（原生 Swing，无需 Node）｜Neovim `Kilo-Org/kilo.nvim`（Neovim 0.11+）

#### Desktop App

#### Web 应用 / 云端 Agent

[https://kilo.ai/cloud](https://kilo.ai/cloud)（每用户隔离 Linux 容器）

#### 移动端 App

[https://kilo.ai/mobile](https://kilo.ai/mobile)（iOS / Android）

#### 服务 / 网关（自托管）

`kilo serve` / `kilo daemon`（Kilo Console 官方标注 Deprecated）

#### 容器镜像（Docker）

#### SDK / 库

`@kilocode/sdk`、`@kilocode/plugin`

**其他说明**：旧 CPU 需用 GitHub Releases 的 `-baseline` 变体（如 `kilo-windows-x64-baseline.zip`）。更新 `kilo upgrade` 或 `npm update -g @kilocode/cli`。可导入 `~/.claude/projects/` 与 `~/.codex/sessions/` 的历史会话。kilocode.ai 已 308 永久重定向至 kilo.ai。

### Continue Dev（第三方）

**官网**：[https://continue.dev](https://continue.dev)（**静态归档页**，官方仓库描述原文 "Static archive of continue.dev (acquired by Cursor)"）｜文档 [https://docs.continue.dev](https://docs.continue.dev)
**GitHub**：`continuedev/continue`（**仍托管完整源码但官方宣布只读**，Apache-2.0，最近仍推送；无独立 CLI 仓库、无公开 hub 仓库）

**作用**：Continue CLI（命令 `cn`）—— 与 VS Code / JetBrains 插件共用同一份 `config.yaml`。

#### Agent CLI（含 TUI）

Windows｜依赖：脚本要求 Node ≥ 20.20.1 + npm，缺失时经 fnm 自动安装 Node

```powershell
irm https://raw.githubusercontent.com/continuedev/continue/main/extensions/cli/scripts/install.ps1 | iex
```

macOS / Linux / WSL / Git Bash｜依赖：curl **或** wget、tar、unzip；脚本自带 fnm 并装 Node 20.20.1，**不依赖系统 Node**；不支持 32 位

```bash
curl -fsSL https://raw.githubusercontent.com/continuedev/continue/main/extensions/cli/scripts/install.sh | bash
```

npm（跨平台）｜依赖：Node.js 20+

```bash
npm i -g @continuedev/cli
```

**安装路径**：CLI 落 npm 全局前缀 `<prefix>/bin`，全局前缀不可写时脚本把 prefix 改为 `~/.npm-global`｜fnm 与托管 Node：macOS / Linux `~/.local/share/fnm`、Windows `%LOCALAPPDATA%\fnm`｜配置 `~/.continue/config.yaml`（Windows `%USERPROFILE%\.continue\config.yaml`）

#### IDE 插件

VS Code `Continue.continue`｜JetBrains [https://plugins.jetbrains.com/plugin/22707-continue](https://plugins.jetbrains.com/plugin/22707-continue)（官方现推荐改用 CLI）

#### Desktop App

#### Web 应用 / 云端 Agent

Web 版当前不可用（hub 页 404）

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`@continuedev/sdk`（EXPERIMENTAL）

**其他说明**：⚠️ 官方 README 顶部声明 **"The `continuedev/continue` repository is no longer actively maintained and is read-only for all users."**（已出 final 2.0.0 release，项目**已被 Cursor 收购**）。GitHub 层面**未 archived**，"只读"是自述而非平台强制。使用前请留意。

### Augment Code（第三方）

**官网**：[https://www.augmentcode.com](https://www.augmentcode.com)　文档 [https://docs.augmentcode.com](https://docs.augmentcode.com)
**GitHub**：`augmentcode/auggie`（**非源码仓库**，仅 issue / 文档 / 示例；许可证原文 "Custom Proprietary License for Augment CLI"，**禁止再分发**）· `augmentcode/auggie-zed-extension`（Zed 扩展官方源码）；**SDK 无独立公开仓库**

**作用**：Auggie CLI（命令 `auggie`）—— 带 Context Engine 语义代码理解、Sub / Parallel Agents。

#### Agent CLI（含 TUI）

全平台｜依赖：**Node.js 20+**（官方安装文档与 npm 包 `engines` 一致）；官方 system requirements 的平台列为 **MacOS、Windows WSL、Linux**

```bash
npm install -g @augmentcode/auggie
```

**安装路径**：可执行由 npm 全局前缀决定｜用户级配置 `~/.augment/settings.json`（Windows `C:\Users\<用户名>\.augment\settings.json`）｜项目级 `<workspace>/.augment/settings.json` 与 `.augment/settings.local.json`｜管理级只读 `/etc/augment/settings.json`（macOS / Linux）、`C:\ProgramData\augment\settings.json`（Windows）

#### IDE 插件

Zed 官方扩展 `augmentcode/auggie-zed-extension`。⚠️ Augment 的 VS Code / JetBrains 扩展属**另一个产品**（官方原文把 IDE agent 与 Auggie 并列），不计入本条目

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

daemon：`npm install -g @augmentcode/auggie@daemon` + `auggie daemon`（会话 `~/.augment/session.json`）

#### 容器镜像（Docker）

#### SDK / 库

`@augmentcode/auggie-sdk`（TS）、`auggie-sdk`（Python）

**其他说明**：官方**只有 npm 一种安装方式**（无 curl / Homebrew / 安装包）。⚠️ Node 版本官方两处不一致：安装文档写 20+，仓库 README 写 22+。收费 标准版 $20/月、Business $100/月。交互模式需支持 ANSI 转义的终端。⚠️ **Windows 原生不在官方支持列表**（仅列 Windows WSL），但配置文档给出了 Windows 路径。

### GitHub（第三方）

**官网**：[https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli)
**GitHub**：`github/copilot-cli`（**非源码仓库**，仅 `.github` / LICENSE / README / changelog / install.sh，**CLI 闭源**）· `github/copilot-sdk`（**独立源码仓库**，六语言，MIT）

**作用**：Copilot CLI（命令 `copilot`）—— Plan / Autopilot 模式；提示词前缀 `&` 可委派云端 agent。

#### Agent CLI（含 TUI）

Windows｜依赖：**Node.js 22+**；**PowerShell 6+**；需有效 Copilot 订阅

```bash
npm install -g @github/copilot
```

`.npmrc` 设了 `ignore-scripts=true` 时：

```bash
npm_config_ignore_scripts=false npm install -g @github/copilot
```

macOS / Linux｜依赖：curl 或 wget；校验用 sha256sum / shasum；**预发布版本额外需要 git**

```bash
curl -fsSL https://gh.io/copilot-install | bash
```

指定版本与目录：

```bash
curl -fsSL https://gh.io/copilot-install | VERSION="v0.0.369" PREFIX="$HOME/custom" bash
```

其他方式

```bash
winget install GitHub.Copilot
brew install --cask copilot-cli
```

**安装路径**：npm / WinGet / Homebrew 各自目录决定｜脚本方式：root 默认 `/usr/local/bin`，非 root 默认 `$HOME/.local/bin`（`PREFIX` 可改）｜配置 `~/.copilot`（Windows `%USERPROFILE%\.copilot`；`COPILOT_HOME` 可改，优先级 `--config-dir` > `COPILOT_HOME` > 默认），含 `settings.json`、`mcp-config.json`、`permissions-config.json`、`session-state/`、`skills/`、`hooks/`｜缓存 Windows `%LOCALAPPDATA%\copilot`

#### IDE 插件

GitHub Copilot 的 IDE 扩展属其他产品形态，不计入本条目

#### Desktop App

#### Web 应用 / 云端 Agent

`copilot --remote` 由 github.com 接管会话；非仓库会话 [https://github.com/copilot/agents](https://github.com/copilot/agents)；`&`（=/delegate）交 Copilot cloud agent 开 draft PR

#### 移动端 App

GitHub Mobile 远程控制（官方 GA 于 "GitHub Mobile and github.com"）

#### 服务 / 网关（自托管）

`copilot --acp --stdio` / `--port 3000`（绑 127.0.0.1）

#### 容器镜像（Docker）

**无官方镜像**（官方原文 "There is no official pre-built Docker image at ghcr.io/github/copilot-cli"）；仅 Dev Container Feature `ghcr.io/devcontainers/features/copilot-cli:1`

#### SDK / 库

`@github/copilot-sdk` 等六语言（Node / Python / Go / .NET / Rust / Java，自动捆绑 CLI）

**其他说明**：预发布另有 `npm install -g @github/copilot@prerelease`、`winget install GitHub.Copilot.Prerelease`、`brew install --cask copilot-cli@prerelease`。首次运行 `/login`；令牌环境变量优先序 `COPILOT_GITHUB_TOKEN` > `GH_TOKEN` > `GITHUB_TOKEN`（需细粒度 PAT + "Copilot Requests" 权限）。

### Amp Labs（第三方）

**官网**：[https://ampcode.com](https://ampcode.com)　文档 [https://ampcode.com/docs](https://ampcode.com/docs)
**GitHub**：**无 Amp 主体公开源码仓库**（`ampcode/amp` 404；SDK 的 repository 字段指向不可公开访问的仓库，许可为 "Amp Commercial License"）· `ampcode/amp.nvim`（**插件源码**，Lua / Apache-2.0）· `ampcode/official-plugins`

**作用**：Amp CLI（命令 `amp`）—— 编码 agent 与开发环境；支持 orbs 云端执行单元、runners、Streaming JSON。

#### Agent CLI（含 TUI）

Windows｜依赖：Windows PowerShell；脚本会做 SHA-256 校验

```powershell
powershell -c "irm https://ampcode.com/install.ps1 | iex"
```

macOS / Linux / WSL｜依赖：curl 或 wget、shasum 或 sha256sum、uname / mktemp / chmod / mkdir / rm（官方文档同时保留「Windows 经 WSL 运行」的口径）

```bash
curl -fsSL https://ampcode.com/install.sh | bash
```

**安装路径**：Windows `%USERPROFILE%\.amp\bin`（`AMP_HOME` 可改）｜macOS / Linux `$AMP_HOME/bin/amp`（默认 `~/.amp/bin`），并在 `~/.local/bin` / `~/bin` / `~/.bin` 中首个在 PATH 可见的目录建软链｜配置 `~/.config/amp/settings.json` 或 `.jsonc`；工作区 `.amp/settings.json`；企业托管文件 macOS `/Library/Application Support/ampcode/managed-settings.json`、Linux `/etc/ampcode/managed-settings.json`

#### IDE 插件

VS Code `sourcegraph.amp`（Marketplace 页面 404，经 API 确认存在）｜Neovim `ampcode/amp.nvim`（JetBrains 已弃用）

#### Desktop App

macOS beta｜依赖：**macOS 26+**

```
https://static.ampcode.com/mac/latest.dmg
```

#### Web 应用 / 云端 Agent

[https://ampcode.com](https://ampcode.com) + orbs（云端执行单元）；runners 自托管：`amp --no-tui [--runner-id <id>]`，落盘 `~/.amp/bin/amp-desktop-helper/<version>`

#### 移动端 App

iOS / iPadOS TestFlight 公开测试｜依赖：iOS / iPadOS 26+（官方 "An App Store release will come later."）

```
https://testflight.apple.com/join/Skjdm6qe
```

#### 服务 / 网关（自托管）

见「Web 应用 / 云端 Agent」的 runners

#### 容器镜像（Docker）

#### SDK / 库

`@ampcode/sdk` / `amp-sdk`

**其他说明**：⚠️ **npm 包已由 `@sourcegraph/amp` 更名为 `@ampcode/cli`**（旧名兼容至 2026-06-15）；官方公告建议改用直接安装脚本。⚠️ 官方文档正文仍保留「经 WSL 运行」的旧措辞，但**同页已正式收录原生 Windows 命令**（该命令由前端动态渲染，直接抓 HTML 看不到）。不支持 Linux riscv64。官方建议 Windows 用 WezTerm / Alacritty 而非 Windows Terminal。收费 Hobby 免费 / Individual $20（每月）。

### Factory AI（第三方）

**官网**：[https://factory.com](https://factory.com)（factory.ai 307 跳转）　文档 [https://docs.factory.com](https://docs.factory.com)
**GitHub**：`Factory-AI/factory`（**非源码仓库**，仅 `.github` / docs / README，无开源许可证，"All rights reserved."）· `Factory-AI/droid-sdk-typescript`、`Factory-AI/droid-sdk-python`（Apache-2.0）· `Factory-AI/examples`（MIT）

**作用**：Factory Droid（命令 `droid` / `droid exec`）—— 终端 AI 编程 Agent，支持 BYOK 与大量托管模型。闭源。

#### Agent CLI（含 TUI）

Windows｜依赖：需 `curl.exe`；架构 x64 / arm64 / x64-baseline（AVX2 检测），SHA256 校验

```powershell
irm https://app.factory.ai/cli/windows | iex
```

macOS / Linux｜依赖：curl（必需）；校验工具可选（缺失仅警告 "skipping checksum verification"）；无需 tar / unzip；Linux 需 `xdg-utils`

```bash
curl -fsSL https://app.factory.ai/cli | sh
```

其他方式

```bash
brew install --cask droid
npm install -g droid
```

**安装路径**：Windows `%USERPROFILE%\bin\droid.exe`（脚本会创建该目录并写用户级 PATH）｜macOS / Linux `$HOME/.local/bin/droid`（脚本**只打印** PATH 建议，不自动改 rc 文件）｜配置 `~/.factory/settings.json`（Windows `%USERPROFILE%\.factory\settings.json`，首次运行自动生成）、覆盖文件 `~/.factory/settings.local.json`、项目级 `<project>/.factory/settings.local.json`｜Spec 默认目录 `~/.factory/specs`｜worktree `~/.factory/worktrees`

#### IDE 插件

VS Code `Factory.factory-vscode-extension`（需先装 CLI）｜JetBrains 2025.3+ 与 Zed 走官方 ACP（JetBrains 配置 `~/.jetbrains/acp.json`）

#### Desktop App

「Factory App」

```
https://app.factory.ai/api/desktop?platform=darwin&architecture=arm64
```

（另有 win32 等平台参数）｜落盘官方未说明

#### Web 应用 / 云端 Agent

[https://app.factory.ai](https://app.factory.ai)（"Nothing to install"）；官方把 Factory App / Droid CLI / Web & Mobile 并列为 Droid 的三个 surface，其中 "Mobile" 仅为手机浏览器

#### 移动端 App

仅手机浏览器版（见上），**无独立 App**

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`@factory/droid-sdk`、`droid-sdk`（Python）

**其他说明**：npm 方式依赖 Node.js >= 20（该包 `engines` 字段）。旧版 `~/.factory/config.json` 仍被加载并与 settings.json 合并（后者优先）。收费 Pro $20 / Plus $100 / Max $200（每月）。

### Charmbracelet（第三方）

**官网**：[https://charm.land/crush](https://charm.land/crush)
**GitHub**：`charmbracelet/crush`（**完整源码仓库**，Go，**许可证为 FSL-1.1-MIT**，不是 MIT）

**作用**：Crush —— 终端 TUI 编码 Agent（Go 单二进制）；多模型（含本地 ollama / lmstudio / llamacpp）、LSP 增强、MCP。

#### Agent CLI（含 TUI）

Windows｜依赖：官方未给 Node 版本要求（npm 包无 `engines`）；**推荐直接用 winget / scoop 装二进制**

```bash
npm install -g @charmland/crush
winget install charmbracelet.crush
scoop bucket add charm https://github.com/charmbracelet/scoop-bucket.git && scoop install crush
```

macOS

```bash
brew install charmbracelet/tap/crush
```

Linux

```bash
yay -S crush-bin
go install github.com/charmbracelet/crush@latest
```

Linux — Debian / Ubuntu 官方 apt 仓库｜依赖：curl、gpg、apt

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://repo.charm.sh/apt/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/charm.gpg
echo "deb [signed-by=/etc/apt/keyrings/charm.gpg] https://repo.charm.sh/apt/ * *" | sudo tee /etc/apt/sources.list.d/charm.list
sudo apt update && sudo apt install crush
```

Linux — Fedora / RHEL 官方 yum 仓库

```bash
echo '[charm]
name=Charm
baseurl=https://repo.charm.sh/yum/
enabled=1
gpgcheck=1
gpgkey=https://repo.charm.sh/yum/gpg.key' | sudo tee /etc/yum.repos.d/charm.repo
sudo yum install crush
```

**安装路径**：可执行由各包管理器决定（npm 包本身是**下载器**，postinstall 才拉平台二进制）｜配置查找顺序 `./.crushrc` → `./crushrc` → `~/.config/crush/crushrc`（Windows `%USERPROFILE%\.config\crush\crushrc`）｜数据 `~/.local/share/crush/crush.json`（Windows `%LOCALAPPDATA%\crush\crush.json`；`CRUSH_GLOBAL_CONFIG` / `CRUSH_GLOBAL_DATA` 可改）｜日志 `./.crush/logs/crush.log`｜全局上下文 `~/.config/crush/CRUSH.md`、`~/.config/AGENTS.md`

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

`crush serve`（多 TUI 共享后端，SSE 事件流、`POST /v1/workspaces`）—— 依据较弱，仅 README 举例

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：FreeBSD 用 `pkg install crush`。Go 方式需 Go ≥ 1.27.0。⚠️ README 提示 `crushrc` 与 `crush.json` **均可执行代码**，须视为可信代码。

### Anomaly（第三方）

**官网**：[https://opencode.ai](https://opencode.ai)
**GitHub**：`anomalyco/opencode`（**完整源码 monorepo**，MIT；`sst/opencode` 会自动重定向至此；v2 在 `tree/v2` 分支）

**作用**：OpenCode —— 开源 AI 编码 Agent，终端 + 桌面（beta）+ Web 三形态；内置 build / plan 两个 Agent。

#### Agent CLI（含 TUI）

Windows｜依赖：**稳定版官方推荐用 WSL**；也提供多种原生方式

```bash
choco install opencode
scoop install opencode
npm install -g opencode-ai
mise use -g github:anomalyco/opencode
```

macOS / Linux｜依赖：tar；脚本自动检测 musl 与无 AVX 环境并切换对应变体；支持 Linux / Darwin / MINGW(Git Bash)

```bash
curl -fsSL https://opencode.ai/v2/install | bash
```

其他方式

```bash
brew install anomalyco/tap/opencode-v2
npm install -g @opencode/cli
bun install -g --trust @opencode/cli
pnpm add -g --allow-build=@opencode/cli @opencode/cli
```

**安装路径**：curl 脚本落 `$HOME/.opencode/bin/opencode`（并写入 shell 配置或 `$GITHUB_PATH`）｜npm / bun / pnpm 由各自全局前缀决定｜全局配置 `~/.config/opencode/opencode.json(c)`；项目级 `opencode.json(c)` 或 `.opencode/opencode.json(c)`｜数据 `~/.local/share/opencode/`（`opencode.db`、`auth.json`、`log/`）｜服务状态 `~/.local/state/opencode/service.json`（可用 `opencode path` 查询各类目录）

#### IDE 插件

VS Code 系（VS Code / Cursor / Windsurf / VSCodium）在集成终端运行 `opencode` 即自动安装｜Zed / JetBrains / Neovim 走 ACP

#### Desktop App

macOS ARM64 / Intel `.dmg`、Windows x64 NSIS、Linux deb / rpm

```bash
brew install --cask opencode-desktop
```

其他平台见 [https://opencode.ai/files/bin/2.0.6/](https://opencode.ai/files/bin/2.0.6/)

#### Web 应用 / 云端 Agent

`opencode web`（本机服务自动开浏览器）；`/share` 生成 opncd.ai 分享页

#### 移动端 App

#### 服务 / 网关（自托管）

`opencode serve`（headless HTTP，OpenAPI 3.1，`OPENCODE_SERVER_PASSWORD`）

#### 容器镜像（Docker）

```bash
docker run -it --rm ghcr.io/anomalyco/opencode
```

#### SDK / 库

`@opencode-ai/sdk`（默认连 `localhost:4096`）

**其他说明**：⚠️ **包名分代** —— 稳定版文档当前推荐 `opencode-ai`（v1，latest 1.18.34），v2 为 `@opencode/cli`（latest 2.0.24）；两包都暴露 `opencode` 命令，**v1 与 v2 默认不能并存**，v2 的 curl 安装器会覆盖 v1 二进制，迁移需先卸载 v1。⚠️ v2 文档写 "Windows package managers are not supported"，而稳定版文档明确列出 choco / scoop / npm 等原生方式——**两版文档口径不同**。官方另注明 "Support for installing OpenCode on Windows using Bun is currently in progress."

### Earendil（第三方）

**官网**：[https://pi.dev](https://pi.dev)
**GitHub**：`earendil-works/pi`（**完整源码 monorepo**，MIT；旧地址 `badlogic/pi-mono` 重定向至此）

**作用**：Pi（命令 `pi`）—— 极简 agent harness；可扩展（TS 扩展、skills、prompts、themes、包目录）；支持 15+ provider。

#### Agent CLI（含 TUI）

Windows｜依赖：**Node.js ≥ 22.19.0 + npm**（缺失时脚本自动从 nodejs.org 下载并校验）；Git Bash 可选

```powershell
powershell -c "irm https://pi.dev/install.ps1 | iex"
```

macOS / Linux｜依赖：curl；preflight 检查 **Node ≥ 22.19 + npm**，缺失时交互式提供 Homebrew / apt / apk / standalone 安装

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

npm（跨平台）

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

Nix

```bash
nix profile add github:earendil-works/pi/stable
```

**安装路径**：Windows 启动器 `%USERPROFILE%\.pi\agent\bin`、托管安装 `%USERPROFILE%\.pi\agent\install`、设置 `.pi\agent\settings.json`、自动装的 Node `%LOCALAPPDATA%\pi-node`｜macOS / Linux Agent 目录 `~/.pi/agent`（`PI_CODING_AGENT_DIR` 可改）、托管安装 `~/.pi/agent/install/releases/<版本>/`、自带 Node `${XDG_DATA_HOME:-~/.local/share}/pi-node/current/bin`｜项目设置 `<项目>/.pi/settings.json`

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`@earendil-works/pi-coding-agent`（`import { createAgentSession }`；Node ≥ 22.19）

**其他说明**：⚠️ 与 oh-my-pi 是**两个不同项目**（oh-my-pi 是 Pi 的 fork），勿混装。`--ignore-scripts` 是官方要求（Pi 不需要依赖生命周期脚本），npm 方式不锁定传递依赖。`pi update` **无法更新 Nix 安装**，需 `nix profile upgrade pi`。

### Stencil Labs（第三方）

**官网**：[https://omp.sh](https://omp.sh)
**GitHub**：`can1357/oh-my-pi`（**完整源码 monorepo**，MIT；Rust crates + 约 16 个 `@oh-my-pi/*` 包 + `sdk/`；无其它官方仓库）

**作用**：oh-my-pi（命令 `omp`，Pi 的下游 fork）—— 31 个内置工具、LSP / DAP 真调试、60+ providers；主打 "Windows-native, skip the WSL"。

#### Agent CLI（含 TUI）

Windows｜依赖：PowerShell ≥ 5.1；**bun ≥ 1.3.14**（缺失时脚本自动安装）；无需 Node

```powershell
irm https://omp.sh/install.ps1 | iex
```

macOS / Linux｜依赖：curl；已有 bun 且满足 ≥ 1.3.14 则走 bun，否则下预编译二进制（无需 bun）；Alpine 需 `apk add libstdc++ libgcc`

```bash
curl -fsSL https://omp.sh/install | sh
```

其他方式

```bash
bun install -g @oh-my-pi/pi-coding-agent
brew install can1357/tap/omp
nix run github:can1357/oh-my-pi
mise use -g github:can1357/oh-my-pi
```

**安装路径**：Windows `%LOCALAPPDATA%\omp\omp.exe`（`PI_INSTALL_DIR` 可改，写入用户级 PATH）｜macOS / Linux `${PI_INSTALL_DIR:-~/.local/bin}/omp`｜设置 `~/.omp/agent/settings.json`｜配置 `~/.omp/agent/config.yml`（默认模型角色）、`~/.omp/agent/models.yml`（自定义 provider）｜会话 `~/.omp/agent/sessions/`；命名 profile `~/.omp/profiles/<name>/agent/sessions/`｜项目配置 `<cwd>/.omp/config.yml`（`PI_CODING_AGENT_DIR`、`PI_CONFIG_DIR`、`OMP_PROFILE` 可改）

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

#### SDK / 库

`@oh-my-pi/pi-coding-agent`（bun ≥ 1.3.14）

**其他说明**：⚠️ **本条目已合并原清单的两条记录** —— `irm https://omp.sh/install.ps1 | iex` 与 `bun install -g @oh-my-pi/pi-coding-agent` 是**同一工具的两种安装方式**。`--source` 会在缺 bun 时自动装 bun（此时另需 git）；`--ref` 源码安装需 `git`（`git-lfs` 可选）。无 npm 安装方式。另有非 8 类形态的 Chrome 扩展「OMP Browser Relay」（`omp browser-relay install` + Load unpacked，落盘 `~/.omp/browser-relay/extension`，**无商店入口**）。

### Gitlawb（第三方）

**官网**：[https://openclaude.gitlawb.com](https://openclaude.gitlawb.com)
**GitHub**：`Twigpine/openclaude`（`Gitlawb/openclaude` 实际解析至此；**完整源码仓库**，含 src / bin / vscode-extension / web）——⚠️ 许可证特殊：LICENSE 实为 NOTICE，**MIT 仅覆盖贡献者修改的部分**，并声明 "Copyright (c) Anthropic PBC. All rights reserved."

**作用**：OpenClaude（命令 `openclaude`）—— 把 Claude Code 式工作流开放给任意 LLM；支持 200+ 模型，配置与 `~/.claude` 完全隔离。

#### Agent CLI（含 TUI）

全平台｜依赖：**Node.js >= 22.0.0**（npm 安装与运行）；建议系统安装 ripgrep（`rg --version` 可见）

```bash
npm install -g @gitlawb/openclaude@latest
```

Arch（社区维护 AUR）

```bash
paru -S openclaude
```

**安装路径**：可执行由 npm 全局前缀决定｜配置 `~/.openclaude` 与 `~/.openclaude.json`（`OPENCLAUDE_CONFIG_DIR` 可改；**`CLAUDE_CONFIG_DIR` 被忽略**）｜后台会话 `~/.openclaude/bg-sessions/`（终止状态在 `bg-sessions/terminal/`）

#### IDE 插件

VS Code 扩展在仓库内（`vscode-extension/openclaude-vscode/`），**无 Marketplace / VSIX 分发**，需 `npm run package` 自打包；依赖 VS Code ≥ 1.95 且 PATH 中有 `openclaude`

#### Desktop App

#### Web 应用 / 云端 Agent

#### 移动端 App

#### 服务 / 网关（自托管）

gRPC 服务：`npm run dev:grpc`（源码运行，默认 `localhost:50051`；官方警告绑定 `0.0.0.0` 且无认证 "not recommended"）

#### 容器镜像（Docker）

#### SDK / 库

**其他说明**：**无官方 curl / Homebrew / 原生安装包**，三平台都只有 npm。**不读取** `~/.claude` 或 `CLAUDE_CONFIG_DIR`，无需预装 Claude Code。与 Anthropic 无隶属或背书关系。源码构建需 Node >= 22.0.0 + Bun >= 1.3.13。README 注明 "OpenClaude does not infer POSIX signal names on Windows"。

### Aider-AI（第三方）

**官网**：[https://aider.chat](https://aider.chat)
**GitHub**：`Aider-AI/aider`（**源码仓库**，Python，Apache-2.0）· `aider-install`、`conventions`、`grep-ast` 及各 benchmark 仓库

**作用**：Aider —— 终端 AI 结对编程（Python 应用），以 git 仓库为单位工作并自动提交。

#### Agent CLI（含 TUI）

Windows｜依赖：脚本先装 uv（内嵌 uv 安装器），再 `uv tool install`；**经 uv 自动准备 Python 3.12**，不要求预装 Python

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://aider.chat/install.ps1 | iex"
```

macOS / Linux｜依赖：curl 或 wget；脚本自带 uv，**不依赖系统 Python**

```bash
curl -LsSf https://aider.chat/install.sh | sh
wget -qO- https://aider.chat/install.sh | sh
```

uv / pipx / pip（跨平台）

```bash
uv tool install --force --python python3.12 --with pip aider-chat@latest
pipx install aider-chat
python -m pip install -U --upgrade-strategy only-if-needed aider-chat
```

aider-install（跨平台）

```bash
python -m pip install aider-install
aider-install
```

**安装路径**：Windows `%USERPROFILE%\.local\bin\aider.exe`（uv 默认；若环境已有 `XDG_BIN_HOME` / `UV_TOOL_BIN_DIR` 会变）｜macOS / Linux `~/.local/bin/aider`｜工具环境：以 `uv tool dir` 实测为准｜配置 `aider.conf.yml` 按 **主目录 → git 仓库根 → 当前目录**顺序查找（后者优先，`--config` 可指定唯一文件）；`.env`、`.aiderignore`、`.aider.input.history`、`.aider.chat.history.md` 默认在 git 根

#### IDE 插件

#### Desktop App

#### Web 应用 / 云端 Agent

浏览器 UI（experimental）：`aider --browser`（`--gui` 为别名）

#### 移动端 App

#### 服务 / 网关（自托管）

#### 容器镜像（Docker）

```bash
docker pull paulgauthier/aider
docker run -it --user $(id -u):$(id -g) --volume $(pwd):/app paulgauthier/aider --openai-api-key $OPENAI_API_KEY
docker pull paulgauthier/aider-full
```

⚠️ 官方文档只说 "available as 2 docker images"，**未使用「官方镜像」措辞**，且位于作者个人命名空间 `paulgauthier/`。容器内无全局 git 配置，需先 `git config user.email` / `user.name`；须在 git 仓库根目录运行。

#### SDK / 库

**其他说明**：官方**明确不建议用系统包管理器**（"they often install aider with incorrect dependencies"）。Python 版本官方两处不一致：安装页写 3.8–3.13，PyPI 元数据为 `>=3.10,<3.13`（pipx / pip 路线为 3.9–3.12）。找不到命令时可用 `python -m aider`。

---

## 三、编排与网关

### Paseo（第三方）

**官网**：[https://paseo.sh](https://paseo.sh)（⚠️ `getpaseo.com` 非官方，勿用）
**GitHub**：`getpaseo/paseo`（**源码仓库**，TS / Apache-2.0；桌面 / 移动 / SDK 均在 monorepo）· `getpaseo/plugins`、`getpaseo/hub`、`getpaseo/paseo-relay`

**作用**：多 Agent 编排工具 —— 守护进程驱动其他 Agent CLI（Claude Code、Codex、Copilot、OpenCode、Pi、Antigravity、Muse Code），**本身不调模型**。

#### Agent CLI（含 TUI）

全平台｜依赖：**需至少一个受支持的 Agent CLI 及其凭据**；建议安装并登录 `gh`；Node 最低版本上游未声明（`.tool-versions` 钉 `nodejs 22.20.0`）

```bash
npm install -g @getpaseo/cli
paseo
```

**安装路径**：可执行由 npm 全局前缀决定｜配置与状态 `~/.paseo`（`PASEO_HOME` 或 `--home` 可改），含 `config.json`、`worktrees/`、`daemon.log`｜默认监听 `ws://127.0.0.1:6767/ws`

#### IDE 插件

#### Desktop App

[https://paseo.sh/download](https://paseo.sh/download)｜依赖：macOS 13+；安装器**自带 daemon 并自动启动**，无需其他组件

```bash
brew install --cask paseo
```

Windows exe（x64 / arm64）｜Linux AppImage / `.deb` / `.rpm`（仅 x86\_64）。AppImage 报 `libfuse.so.2` 时：

```bash
sudo apt install libfuse2t64
./Paseo-x86_64.AppImage --appimage-extract-and-run
```

#### Web 应用 / 云端 Agent

[https://app.paseo.sh](https://app.paseo.sh)；daemon 也可自托管同一 UI（需 `PASEO_PASSWORD`）

#### 移动端 App

iOS [https://apps.apple.com/app/paseo-pocket-engineer/id6758887924](https://apps.apple.com/app/paseo-pocket-engineer/id6758887924)｜Android [https://play.google.com/store/apps/details?id=sh.paseo](https://play.google.com/store/apps/details?id=sh.paseo)（另发 APK）｜需与 daemon 配对

#### 服务 / 网关（自托管）

```bash
docker run -d --name paseo -p 6767:6767 -e PASEO_PASSWORD=change-me -v "$PWD/paseo-home:/home/paseo" -v "$PWD:/workspace" ghcr.io/getpaseo/paseo:latest
```

访问 `http://localhost:6767`；容器内状态在 `/home/paseo`、工作区 `/workspace`

#### 容器镜像（Docker）

`ghcr.io/getpaseo/paseo:latest`｜⚠️ 镜像**不内置任何 agent CLI**，需自行添加

#### SDK / 库

`@getpaseo/client`

**其他说明**：内置 E2EE relay 便于配对（可关闭）。守护进程内置 MCP server（create\_agent、send\_agent\_prompt、get\_agent\_status 等），可注入被启动的 Agent。提供编排 Skills（`/paseo`、`/paseo-handoff` 等）。个人维护项目。

### OpenClaw Foundation（第三方）

**官网**：[https://openclaw.ai](https://openclaw.ai)　文档 [https://docs.openclaw.ai/](https://docs.openclaw.ai/)
**GitHub**：`openclaw/openclaw`（**源码仓库**，MIT）· `openclaw/openclaw-windows-node`（**Windows Hub 源码仓库**，.NET / WinUI，含 `installer.iss`）· `openclaw/homebrew-tap`、`openclaw/clawhub`

**作用**：开源自托管的 AI 助手 / 代理网关，接入 WhatsApp、Telegram、Discord、Slack、Teams、iMessage、Signal 等聊天 App。MIT，无企业版、无付费版。

#### Agent CLI（含 TUI）

Windows｜依赖：**Node 24.16+ 或 26.1+**；⚠️ **Git 必需**（即使 npm 安装）；PowerShell 5+

```powershell
powershell -c "irm https://openclaw.ai/install.ps1 | iex"
```

macOS / Linux / WSL2｜依赖：macOS 15+ / Linux x64；Node 24.16.0+ 或 26.1.0+（不满足会经 Homebrew 或 apt / apk 自动安装；macOS 首次可能要求管理员密码装 Homebrew）

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

仅 CLI（跳过引导，**自带用户态 Node 24.21.0，不需要系统 Node**）

```bash
curl -fsSL https://openclaw.ai/install-cli.sh | bash
```

npm / pnpm（⚠️ npm 12 必须加 `--allow-scripts=openclaw`）

```bash
npm i -g openclaw
pnpm add -g --allow-build=openclaw openclaw@latest
```

**安装路径**：状态目录 `~/.openclaw`（`OPENCLAW_STATE_DIR`）；主配置 `~/.openclaw/openclaw.json`（JSON5，`OPENCLAW_CONFIG_PATH` 可改）｜工作区 `~/.openclaw/workspace`（`agents.defaults.workspace` 可改），含 `AGENTS.md`、`SOUL.md`、`IDENTITY.md`、`USER.md`、`MEMORY.md`、`memory/`｜`~/.openclaw/credentials/`、`~/.openclaw/state/openclaw.sqlite`、`~/.openclaw/agents/<agentId>/`｜shell 安装的入口 `~/.local/bin/openclaw`（npm 方式由 npm 前缀决定，必要时脚本把前缀切到 `~/.npm-global`）｜自定义 profile 为 `~/.openclaw-<profile>/`

#### IDE 插件

#### Desktop App

macOS `OpenClaw-<version>.dmg`（preferred，15+）或 `.zip`｜Windows `OpenClawCompanion-Setup-x64.exe` / `-arm64.exe`｜Linux `.deb` / `.AppImage`（glibc 2.35+）——均在 [https://github.com/openclaw/openclaw/releases](https://github.com/openclaw/openclaw/releases)｜落盘：macOS 仅注明 "Device preferences stay on this Mac"，其余官方未说明

#### Web 应用 / 云端 Agent

Web Control UI 与 WebChat：`http://127.0.0.1:18789/`（可装为 PWA）｜⚠️ 官方明确**无托管云版本**

#### 移动端 App

iOS [https://apps.apple.com/app/openclaw-ai-that-does-things/id6780396132](https://apps.apple.com/app/openclaw-ai-that-does-things/id6780396132)｜Android [https://play.google.com/store/apps/details?id=ai.openclaw.app](https://play.google.com/store/apps/details?id=ai.openclaw.app)｜**需运行中的 Gateway**

#### 服务 / 网关（自托管）

```bash
openclaw onboard --install-daemon
openclaw gateway install
```

macOS 用 LaunchAgent、Linux / WSL2 用 systemd、Windows 用计划任务。维护：`openclaw update`、`openclaw update --channel dev`、`openclaw doctor`

#### 容器镜像（Docker）

`ghcr.io/openclaw/openclaw:latest` 与 `openclaw/openclaw:latest`｜落盘 `/home/node/.openclaw`

#### SDK / 库

Plugin SDK + `@openclaw/gateway-client` / `@openclaw/gateway-protocol`（均来自主仓库 `packages/gateway-client`，非独立仓库）

**其他说明**：Beta 通道 `curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta`；源码安装 `... | bash -s -- --install-method git`（需 git、corepack、pnpm）。脚本会校验 node:sqlite 能力（"SQLite 3.51.3+, 3.50.7+ within 3.50.x, or 3.44.6+ within 3.44.x is required"）。终端安装器支持 Debian、Ubuntu、Fedora、Arch、Raspberry Pi OS。iOS 独立仓库未确认。