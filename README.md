# AI Playbook

> AI 助手行为约束规范与 Skills 技能集合。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 仓库简介

本仓库收录了一套面向 AI 编程助手的**行为约束规范**与**可复用技能（Skills）**，目标是让 AI 助手在软件开发任务中输出稳定、规范、安全的结果。

- **行为规范**：[`ai-assistant-behavior-rules.md`](ai-assistant-behavior-rules.md) — 定义 AI 助手的语言要求、环境变量管理、文件系统边界、依赖管理、隐私与安全底线等行为准则。
- **Skills**：[`skills/`](skills/) — 可复用技能包，以 Claude Code Skills 格式组织，覆盖代码审查、数据库设计、功能编码等高频开发场景。

## 目录结构

```
.
├── ai-assistant-behavior-rules.md   # AI 助手行为约束规范（全局规则）
├── skills/                          # AI 助手 Skills 技能包
│   ├── code-review/                 # 代码审查
│   ├── db-design-standard/          # 数据库设计规范
│   ├── feature-coding-standard/     # 功能模块开发规范
│   └── method-ordering/             # 方法（接口）排序规则
└── LICENSE                          # MIT 许可证
```

## Skills 一览

| Skill | 用途 |
| --- | --- |
| `code-review` | 审查 Java 代码是否符合最佳实践、是否存在安全问题，以及是否遵循 Spring Framework 规范 |
| `db-design-standard` | 数据库脚本设计与规范检查，覆盖建表、结构变更与 SQL 生成 |
| `feature-coding-standard` | 功能模块开发规范与代码风格指南 |
| `method-ordering` | 统一接口/方法的排序规则（Controller/Service/Mapper/Feign/@HttpExchange） |

## 使用方法

### 在 Claude Code 中使用 Skills

Skills 采用 Claude Code 的 `SKILL.md` 格式编写，可直接复制到目标项目的 `.claude/skills/` 下使用：

```bash
# 在项目根目录执行
cp -r <本仓库路径>/skills/* .claude/skills/
```

之后在会话中即可通过 `/code-review`、`/db-design-standard` 等命令直接调用。

> 其他 AI 助手（如 Cursor、Copilot 等）可将各 SKILL.md 的内容作为规范文档，配置到对应的规则（Rules）机制中使用。

### 应用行为规范

将 [`ai-assistant-behavior-rules.md`](ai-assistant-behavior-rules.md) 的内容合并到全局 `~/.claude/CLAUDE.md` 或项目的 `CLAUDE.md` 中，即可让 AI 助手遵守对应的行为约束。

## 贡献

欢迎通过 Issue 或 Pull Request 提出改进建议。提交信息请遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范。

## 许可证

[MIT](LICENSE)
