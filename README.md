# AI Handbook

> 面向 AI 编程助手的行为约束规范与可复用 Skills 技能库。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 目录

- [仓库简介](#仓库简介)
- [快速开始](#快速开始)
- [目录结构](#目录结构)
- [Skills 一览](#skills-一览)
- [使用方法](#使用方法)
  - [安装 Skills](#安装-skills)
  - [应用通用规范](#应用通用规范)
- [贡献](#贡献)
- [许可证](#许可证)

## 仓库简介

本仓库收录面向 AI 编程助手的**行为约束规范**与**可复用技能（Skills）**，旨在使 AI 助手在软件开发任务中输出稳定、规范、安全的结果。

| 模块 | 位置 | 说明 |
| --- | --- | --- |
| 通用行为规范 | [`behavior-rules.md`](behavior-rules.md) | 约定语言要求、环境变量管理、软件与工具安装、隐私与安全要求、沟通与执行方式等行为准则 |
| 通用开发规范 | [`dev-rules.md`](dev-rules.md) | 约定异常信息、日志、提示语及代码输出原则上使用英文，代码注释以中文为主并保留专业英文术语；同时约定文档注释格式、文件系统边界、依赖管理、脚本管理、版本控制、交付验证与密钥管理要求 |
| 工具清单 | [`catalog/tools.md`](catalog/tools.md) | 收录 Windows / macOS / Linux 三平台各 AI 编程工具的官方安装命令、依赖环境与默认安装路径，命令与路径均取自官方文档或官方安装脚本；一级按「本地模型运行时 / 编程工具 / 编排与网关」分类，二级为出品方，三级为发行形态（Agent CLI / IDE 插件 / Desktop App / Web / 移动端 / 服务网关 / 容器镜像 / SDK） |
| 技能清单 | [`catalog/skills.md`](catalog/skills.md) | 收录外部 Agent Skills 的安装清单，含 `npx skills` 安装命令、全局 / 项目级部署层级建议及各 skill 功能说明 |
| 插件清单 | [`catalog/plugins.md`](catalog/plugins.md) | 收录各 AI agent 工具的插件与插件市场；目前已收录 Claude Code，Codex 与 Pi 待补充 |
| 指令文件模板 | [`references/`](references/) | 提供 `AGENTS.md` 与 `CLAUDE.md` 两份等效模板；写入 agent 指令文件后，AI 助手将在执行任务前自动获取并遵循最新规范（见[应用通用规范](#应用通用规范)） |
| Skills 技能包 | [`skills/`](skills/) | 本仓库提供的 5 个可复用技能，遵循开放的 Agent Skills（`SKILL.md`）规范组织，兼容 Claude Code、Codex、Cursor、Pi 等主流 AI 编程助手（见[Skills 一览](#skills-一览)） |

> [`catalog/`](catalog/) 收录外部生态资源的用途与安装方式等信息，与本仓库自有的规则、技能包相区分；其中 `catalog/skills.md` 与 [`skills/`](skills/) 目录名称相近，使用时请注意区分。

## 快速开始

**1. 安装 Skills**（推荐方式，支持符号链接与更新管理）：

```bash
npx skills add cnzeropro/ai-handbook
```

安装完成后重启 agent 会话，即可通过 `/java-code-review`、`/db-design-standard` 等命令调用相应技能。其他安装方式见[安装 Skills](#安装-skills)。

**2. 应用通用规范**：将 [`references/AGENTS.md`](references/AGENTS.md) 的内容写入 agent 的用户级指令文件（如 `~/.codex/AGENTS.md`、`~/.claude/CLAUDE.md`），即可一次配置、全局生效。安装命令与 AI 助手提示词见[应用通用规范](#应用通用规范)。

## 目录结构

```text
.
├── behavior-rules.md                # 通用行为规范（全局规则）
├── dev-rules.md                     # 通用开发规范（输出语言 / 注释 / 文件系统 / 依赖 / 脚本 / 版本控制 / 交付验证 / 密钥）
├── catalog/                         # 外部资源清单（工具 / 技能 / 插件）
│   ├── tools.md                     # AI 编程工具清单（三平台官方安装命令 / 依赖环境 / 安装路径）
│   ├── skills.md                    # 外部 Agent Skills 安装清单（npx skills）
│   └── plugins.md                   # AI agent 插件与插件市场清单
├── skills/                          # AI 编程助手 Skills 技能包（本仓库自带）
│   ├── java-coding-standard/        # Java 编码规范（语言级）
│   ├── java-feature-standard/       # 功能模块开发规范（模块级）
│   ├── java-method-ordering/        # 方法（接口）排序规则
│   ├── java-code-review/            # 代码审查
│   └── db-design-standard/          # 数据库设计规范
├── references/                      # 指令文件模板（AGENTS.md / CLAUDE.md）
└── LICENSE                          # MIT 许可证
```

## Skills 一览

| Skill | 用途 |
| --- | --- |
| `java-coding-standard` | Java 语言级编码规范：命名、注释、格式与类成员布局、常量与枚举、异常与错误码、日志、工具类（JDK/Spring/Hutool/Guava/Commons）、Java 版本适配 |
| `java-feature-standard` | 功能模块开发规范：分层架构、命名与对象模型（PO(Persistent)/DTO/PO(Param)/VO）、注解与事务、接口设计（校验/幂等/错误响应）、二方库与依赖管理 |
| `java-method-ordering` | 接口/端点/契约方法的排序规则：五类语义排序（查询/新增/更新/删除/其他）+ 组内细化（Controller/Service/Mapper/Feign/@HttpExchange 等各层落地） |
| `java-code-review` | Java 代码审查：安全（含深度清单）、框架最佳实践、代码质量、性能、并发、持久层、日志 + Java 语言陷阱清单，附三级问题输出模板 |
| `db-design-standard` | 关系型数据库表设计与 SQL 脚本规范：10 条设计规则、SQL 编写规约、辅助索引设计指南、DDL 变更规范、多库适配映射与验收清单 |

> 以上五个 skill 各司其职：`java-coding-standard` 约束语言级编码风格，`java-feature-standard` 约束模块级结构设计，`java-method-ordering` 约束方法组织顺序，`java-code-review` 提供系统化的审查清单，`db-design-standard` 规范库表设计与 SQL 编写。

## 使用方法

### 安装 Skills

本仓库技能遵循开放的 Agent Skills（`SKILL.md`）规范编写，可通过以下任一方式安装：

**方式一：`npx skills`（推荐，支持符号链接与更新管理）**

```bash
# 从 GitHub 仓库安装（交互式选择 Symlink 或 Copy；CLI 会自动检测已安装的 agent）
npx skills add cnzeropro/ai-handbook

# 或从本地路径安装
npx skills add <本仓库路径>

# 仅安装指定 skill / 安装到用户级（跨项目）
npx skills add cnzeropro/ai-handbook --skill db-design-standard
npx skills add cnzeropro/ai-handbook -g

# 指定目标 agent（默认全装到检测到的 agent；支持 claude-code / codex / cursor / pi / opencode 等 25+ 种）
npx skills add cnzeropro/ai-handbook -a claude-code -a codex -a pi
```

**方式二：直接复制（复制至目标 agent 的对应目录，目标目录需已存在）**

| Agent | 项目级目录 | 用户级目录（跨项目） |
| --- | --- | --- |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.codex/skills/` | `~/.codex/skills/` |
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| Pi | `.pi/skills/` | `~/.pi/agent/skills/` |

```bash
# 示例：安装到 Claude Code（项目级）
cp -r <本仓库路径>/skills/* .claude/skills/

# 示例：安装到 Codex（用户级，跨项目复用）
cp -r <本仓库路径>/skills/* ~/.codex/skills/
```

**方式三：交给 AI 助手安装（提示词）**

将以下提示词发送给任意 AI 编程助手，由其深度扫描本机已安装的全部 AI agent 工具并代为安装：

```text
请深度扫描本机已安装的所有 AI 编程助手（AI agent）工具，包括但不限于 Claude Code、Codex、Cursor、Pi、OpenCode、Gemini CLI 等，注意不要遗漏任何工具。

然后为本仓库（GitHub：cnzeropro/ai-handbook）安装 Skills 技能包：
1. 优先执行 npx skills add cnzeropro/ai-handbook，由 CLI 自动检测已安装的 agent 并完成安装；
2. 若存在 npx skills 未覆盖的工具，先克隆该仓库（或下载 skills/ 目录），再将各技能包（含 references/ 子目录）复制到该工具的用户级技能目录（优先用户级，以便跨项目复用）；
3. 采用覆盖模式安装或更新同名 skill；
4. 完成后输出清单：每个工具的名称、安装方式、目标路径与结果（成功 / 已覆盖 / 不支持 / 失败原因）。
```

安装完成后重启对应 agent 会话，即可通过 `/java-code-review`、`/db-design-standard` 等命令调用相应技能（各 skill 的 `references/` 目录将一并安装，并按需自动加载）。

> 对于未直接支持 skills 机制的 AI 助手（如 Copilot Chat 等），可将各 SKILL.md 的内容作为规范文档，配置至其对应的规则（Rules）机制中使用。

### 应用通用规范

[`references/`](references/) 下的 `AGENTS.md` 与 `CLAUDE.md` 内容等效（前者为跨 agent 通用格式）。写入 agent 指令文件后，AI 助手将在执行任务前自动获取并遵循最新规范（一级 [`behavior-rules.md`](behavior-rules.md)，二级 [`dev-rules.md`](dev-rules.md)，涵盖行为规范与开发规范两部分）。亦可直接将两份规范文件合并至现有指令文件（完全离线可用，但规范更新后需手动重新合并）。

**推荐配置至全局（用户级）**，一次配置对所有项目生效；亦可仅部署至单个项目：

| Agent | 全局（推荐） | 项目级（可选） |
| --- | --- | --- |
| Claude Code | `~/.claude/CLAUDE.md` | `<项目>/AGENTS.md`（v2.1.277+，需项目内无 `CLAUDE.md`） |
| Codex | `~/.codex/AGENTS.md` | `<项目>/AGENTS.md` |
| Cursor | 设置 → Rules → User Rules | `<项目>/AGENTS.md` |
| Pi | `~/.pi/agent/AGENTS.md` | `<项目>/AGENTS.md` |

> Claude Code 的 `AGENTS.md` 仅在项目级生效，且默认在项目已有 `CLAUDE.md` 时整体失效（可在 `/config` → Project instructions 切换为两者并用）；用户级仍只有 `~/.claude/CLAUDE.md`。

以下以 Codex 全局配置文件为例，提供覆盖、前插、追加三种写法，任选其一即可（目标目录需已存在）；适配其他 agent 时，仅需替换命令中的目标路径。Claude Code 用户级请改用 `references/CLAUDE.md` 与 `~/.claude/CLAUDE.md`。

**macOS / Linux / Git Bash**

```bash
# 覆盖
P=~/.codex/AGENTS.md; curl -fsSL https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md -o "$P"

# 追加到开头
P=~/.codex/AGENTS.md; touch "$P" && { curl -fsSL https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md; printf '\n'; cat "$P"; } > "$P.new" && mv "$P.new" "$P"

# 追加到末尾
P=~/.codex/AGENTS.md; { printf '\n'; curl -fsSL https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md; } >> "$P"
```

**Windows（PowerShell 7+）**

```powershell
# 注：PowerShell 5.1 中 curl 是 Invoke-WebRequest 的别名，故统一使用 curl.exe

# 覆盖
$p = "$home\.codex\AGENTS.md"; curl.exe -fsSL https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md -o $p

# 追加到开头
$p = "$home\.codex\AGENTS.md"; curl.exe -fsSL https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md -o "${p}.new"; Set-Content $p -Encoding utf8 -Value ((Get-Content "${p}.new" -Raw) + "`n" + (Get-Content $p -Raw -ErrorAction SilentlyContinue)); Remove-Item "${p}.new" -Force

# 追加到末尾
$p = "$home\.codex\AGENTS.md"; curl.exe -fsSL https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md -o "${p}.new"; Add-Content $p -Encoding utf8 -Value ("`n" + (Get-Content "${p}.new" -Raw)); Remove-Item "${p}.new" -Force
```

**交给 AI 助手安装（提示词）**

不想逐个执行命令时，可将以下提示词发送给任意 AI 编程助手，由其深度扫描本机已安装的全部 AI agent 工具，并将全局规则以覆盖模式安装到每一个支持的工具：

```text
请深度扫描本机已安装的所有 AI 编程助手（AI agent）工具，包括但不限于 Claude Code、Codex、Cursor、Pi、OpenCode、Gemini CLI 等，注意不要遗漏任何工具。

然后将本仓库通用规范（指令文件模板）安装到每一个支持全局规则（用户级指令文件）的工具，采用覆盖模式：
1. 下载 https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/references/AGENTS.md；
2. 按各工具的官方约定写入用户级指令文件，如 Claude Code 写入 ~/.claude/CLAUDE.md，Codex 写入 ~/.codex/AGENTS.md，Pi 写入 ~/.pi/agent/AGENTS.md，其余工具依此类推；
3. 覆盖前，若目标文件已存在，先备份为 <原路径>.bak；
4. 不支持全局规则文件的工具（如仅支持在设置界面配置的 Cursor），跳过并在清单中说明原因；
5. 完成后输出清单：每个工具的名称、目标路径与结果（已覆盖 / 已备份 / 跳过原因 / 失败原因）。
```

## 贡献

欢迎通过 Issue 或 Pull Request 提交改进建议，提交信息请遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范。

## 许可证

[MIT](LICENSE)
