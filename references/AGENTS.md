# 一级规则（最重要）

## 规则获取与缓存（各级规则通用）

本文件所有规则均按下述机制获取与缓存：

- **缓存位置**：系统临时目录 + 来源仓库子目录 + 原文件名；下载时直接覆盖同名旧文件（不改名、不另存副本）。来源仓库名取 URL 中的仓库段（如 `ai-handbook`、`andrej-karpathy-skills`）：
  - Windows 示例：`%TEMP%\ai-handbook\behavior-rules.md`、`%TEMP%\andrej-karpathy-skills\CLAUDE.md`（PowerShell 写作 `$env:TEMP\`）
  - Linux/macOS 示例：`${TMPDIR:-/tmp}/ai-handbook/behavior-rules.md`（`TMPDIR` 未设置时即 `/tmp`）
- **时效判断**：以缓存文件的修改时间为准；24 小时内直接复用，过期或无法确认时效的重新获取并覆盖。
- **获取方式**（按序尝试，任一成功即可；`<URL>`、`<文件名>` 按当前规则文件替换）：
  1. 命令下载（首选，内容原样）：三平台基本自带 curl（Windows 10 1803+/11 自带 `curl.exe`；macOS 与主流 Linux 自带 `curl`；PowerShell 5.1 中 `curl` 是别名，须写作 `curl.exe`）：
     - Linux/macOS：`curl -fsSL --create-dirs "<URL>" -o "${TMPDIR:-/tmp}/ai-handbook/<文件名>"`
     - Windows（PowerShell）：`curl.exe -fsSL --create-dirs "<URL>" -o "$env:TEMP\ai-handbook\<文件名>"`
     - Windows（CMD）：`curl.exe -fsSL --create-dirs "<URL>" -o "%TEMP%\ai-handbook\<文件名>"`
  2. Windows 直连失败（如连接被重置）时，改用 PowerShell 的 `iwr`（自动使用系统代理）：先 `New-Item -ItemType Directory -Force "$env:TEMP\ai-handbook" | Out-Null`，再 `iwr "<URL>" -OutFile "$env:TEMP\ai-handbook\<文件名>" -UseBasicParsing`。
  3. 无法执行命令或无法写文件时，改用内置联网工具（如 WebFetch）获取。
- 全部失败时，可询问用户是否有可用代理，临时设置后重试（如 `HTTPS_PROXY`，仅当前命令/会话有效，见通用行为规范「网络与代理」）；仍失败须暂停并告知用户，不得在未参照规范的情况下行动。

## 通用行为规范

**所有行动之前，都必须先参照并遵守本规范。**

规范文件：[behavior-rules.md](https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/behavior-rules.md)

要求：

1. 处理任何任务之前，先按上述机制获取该文件的最新内容并完整通读；
2. 此后所有行动（回复、运行命令、读写文件等）均须严格遵循该规范的约束。

# 二级规则

## 通用开发规范

**在遵守上述规范的基础上，当任务涉及项目开发（编写、修改代码或脚本，操作项目文件与依赖）时，同时参照并遵守本规范。**

规范文件：[dev-rules.md](https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/dev-rules.md)

要求：

1. 获取、缓存、通读与遵守要求同上；
2. 本规范与上述规范冲突时，以上述规范为准。

## andrej-karpathy-skills 编码行为准则

**在遵守上述规范的基础上，当任务涉及编码（读写代码）时，同时参照并遵守本准则（减少 LLM 常见编码错误）。**

准则文件：[andrej-karpathy-skills/CLAUDE.md](https://raw.githubusercontent.com/multica-ai/andrej-karpathy-skills/main/CLAUDE.md)

要求：

1. 获取、缓存、通读、遵守及冲突处理要求同上。
