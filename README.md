# AI Handbook

> AI 助手行为约束规范与 Skills 技能集合。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 仓库简介

本仓库收录了一套面向 AI 编程助手的**行为约束规范**与**可复用技能（Skills）**，目标是让 AI 助手在软件开发任务中输出稳定、规范、安全的结果。

- **行为规范**：[`rules.md`](rules.md) — 定义 AI 助手的语言要求、环境变量管理、文件系统边界、依赖管理、隐私与安全底线等行为准则。
- **工具清单**：[`tools.md`](tools.md) — 收录 **Windows / macOS / Linux** 三平台下各 AI 编程工具的官方安装命令、依赖环境与默认安装路径。一级按「本地模型运行时 / 编程工具 / 编排与网关」分类，二级为出品方，三级为发行形态（Agent CLI / IDE 插件 / Desktop App / Web / 移动端 / 服务网关 / 容器镜像 / SDK）；命令与路径均取自官方文档或官方安装脚本原文。
- **技能清单**：[`skills.md`](skills.md) — 外部 Agent Skills 的安装清单：`npx skills` 命令、全局 / 项目级的层级建议，以及每个 skill 的作用。注意与下方 `skills/`（本仓库自带的技能包）区分。
- **插件清单**：[`plugins.md`](plugins.md) — 各 AI 客户端的插件与插件市场，目前收录 Claude Code，结构已预留 Codex / Pi 等客户端的扩展位置。
- **指令文件模板**：[`references/`](references/) — 提供 `CLAUDE.md` 与 `AGENTS.md` 两份等效模板，复制到项目或全局配置后，AI 助手即可在行动前加载并遵守上述行为约束规范。
- **Skills**：[`skills/`](skills/) — 可复用技能包，以开放的 Agent Skills（`SKILL.md`）格式组织，兼容 Claude Code、Codex、Cursor、Pi 等主流 AI 编程助手，覆盖代码审查、数据库设计、功能编码等高频开发场景。

## 目录结构

```
.
├── rules.md                         # AI 助手行为约束规范（全局规则）
├── tools.md                         # AI 编程工具清单（三平台官方安装命令 / 依赖环境 / 安装路径）
├── skills.md                        # 外部 Agent Skills 安装清单（npx skills）
├── plugins.md                       # AI 客户端插件与插件市场清单
├── skills/                          # AI 助手 Skills 技能包（本仓库自带）
│   ├── java-coding-standard/        # Java 编码规范（语言级）
│   ├── java-feature-standard/       # 功能模块开发规范（模块级）
│   ├── java-method-ordering/        # 方法（接口）排序规则
│   ├── java-code-review/            # 代码审查
│   └── db-design-standard/          # 数据库设计规范
├── references/                      # 指令文件模板（CLAUDE.md / AGENTS.md）
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

> 五个 skill 的分工：`java-coding-standard` 管"语言怎么写"，`java-feature-standard` 管"模块怎么搭"，`java-method-ordering` 管"方法怎么排"，`java-code-review` 管"问题怎么查"，`db-design-standard` 管"库表怎么建"。

## 使用方法

### 在 Claude Code 中使用 Skills

Skills 采用 Claude Code 的 `SKILL.md` 格式编写，可通过以下任一方式安装：

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

**方式二：直接复制（按目标 agent 的目录）**

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

安装后重启对应 agent 会话即可通过 `/java-code-review`、`/db-design-standard` 等命令直接调用（各 skill 的 `references/` 目录会一并安装，按需自动加载）。

> 未直接支持 skills 机制的 AI 助手（如 Copilot Chat 等）可将各 SKILL.md 的内容作为规范文档，配置到对应的规则（Rules）机制中使用。

### 应用行为规范

将 [`rules.md`](rules.md) 的内容合并到全局 `~/.claude/CLAUDE.md` 或项目的 `CLAUDE.md` 中，即可让 AI 助手遵守对应的行为约束。

也可以将 [`references/`](references/) 下的模板复制到项目根目录或全局配置：Claude Code 使用 `CLAUDE.md`，其他 agents（Codex、Cursor 等）使用 `AGENTS.md`，两者内容等效；模板会让 AI 助手在行动前通过 GitHub 地址获取最新规范并遵守。

## 贡献

欢迎通过 Issue 或 Pull Request 提出改进建议。提交信息请遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范。

## 许可证

[MIT](LICENSE)
