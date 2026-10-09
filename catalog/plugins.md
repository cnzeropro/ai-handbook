# AI 插件清单

> 收录各 AI agent 工具的**插件（Plugin）与插件市场（Marketplace）**。结构约定：一级为 AI agent 工具，二级为插件市场，三级为插件；无插件市场概念的工具，下层级依次上提。目前收录 Claude Code，Codex、Pi 待补充。

## Claude Code

**插件说明**

- **Plugin**：把 skills / agents / hooks / MCP servers / commands / LSP / 主题等组件打包成的可安装单元，用于统一分发与管理
- **Marketplace**：包含 `.claude-plugin/marketplace.json` 的仓库或目录，登记可用插件及其获取位置

**安装层级（scope）**

| scope | 写入的 settings 文件 | 生效范围 |
| --- | --- | --- |
| `user`（`install` 默认） | `~/.claude/settings.json` | 本机所有项目 |
| `project` | `.claude/settings.json`（提交进仓库） | 仓库内所有人 |
| `local` | `.claude/settings.local.json`（gitignore） | 仅你在本仓库 |

- 项目 scope **只提交 `enabledPlugins` 条目，不会替协作者下载插件** —— 每人仍需各自 `claude plugin install <plugin>@<marketplace> --scope project`。
- `managed` 不是安装 scope：组织通过策略文件统一管控，它只在 `claude plugin update --scope` 中可用。
- 同一插件存在于多个 scope 时优先级：`local` > `project` > `user`。
- **落盘目录**：插件根目录 `~/.claude/plugins`（可用 `CLAUDE_CODE_PLUGIN_CACHE_DIR` 改），下含 `cache/<marketplace>/<plugin>/<version>/`、`marketplaces/<name>/`、`data/<plugin-id>/`、`installed_plugins.json`、`known_marketplaces.json`。

**常用命令**

```bash
# 市场
claude plugin marketplace add <owner/repo | URL | 本地路径>
claude plugin marketplace list
claude plugin marketplace update [name]        # 只刷新市场目录，不更新已装插件版本
claude plugin marketplace remove <name>        # 会一并卸载从该市场安装的插件

# 插件
claude plugin install <plugin>@<marketplace> [--scope user|project|local]
claude plugin list [--json]
claude plugin update <plugin> [-y]
claude plugin uninstall <plugin> [--keep-data]
claude plugin disable <plugin> | claude plugin disable --all
claude plugin enable <plugin>
```

会话内等价形式（`/plugin`、`/plugins`、`/marketplace` 互为别名）：

```text
/plugin                              打开面板（Discover 标签）
/plugin list                         列出已装插件
/plugin install <插件>               打开该插件详情
/plugin marketplace add <source>     添加市场
/reload-plugins [--force]            把待生效的变更应用到当前会话
```

一次性「添加市场并安装插件」（需 v2.1.275+）：

```bash
/plugin install <plugin> --marketplace <source>
```

**注意事项**

- ⚠️ **插件会增加上下文开销**：启用后，其每个 skill / agent / command 的**名称与描述会进入每一轮上下文**（正文只在真正使用时加载）。`claude plugin details <name>` 可查看 `Always-on` token 数与逐组件成本。
- ⚠️ 市场名 `claude-plugins-official`、`anthropic-agent-skills` 等**官方名称被保留**，第三方市场使用这些名称会报 `The name '<name>' is reserved for official Anthropic marketplaces`。
- 官方**不维护「市场内插件清单」文档页**（"The catalog changes often, so this page doesn't list it"），最新清单可通过会话内 `/plugin` 的 **Discover** 标签或网页版 <https://claude.com/marketplace/plugins> 查看。
- `marketplace remove` 用 **`marketplace.json` 里的 `name` 字段**，不是执行 `add` 时传入的 source（两者可能不同，例如 `anthropics/skills` 的市场名是 `anthropic-agent-skills`）。
- 卸载时若该插件仍被其他已启用的插件依赖，命令会被拒绝并给出链式卸载命令。
- 从**本地路径**添加的市场中的插件是就地加载，修改源码后下次会话即生效，**无需提升版本号**；其他市场插件会复制到 `~/.claude/plugins/cache/` 后加载。
- 云会话（含网页版 claude.ai/code）**不加载**本地设置中的插件。

以下按插件市场列出本清单收录的插件。

### [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)

- **提供方**：Anthropic
- **市场名**：`claude-plugins-official`
- **规模**：315 个插件条目——Anthropic 自研 39 个（含 15 个语言服务器插件）、合作方内嵌 14 个、远程引用 262 个
- **特性**：首次启动交互式会话时自动添加，无需手动 `add`；默认开启自动更新（第三方市场默认关闭）

仅在从未运行过交互式会话的机器上（如 CI 环境）需要显式添加：

```bash
claude plugin marketplace add anthropics/claude-plugins-official
```

#### `superpowers`

- **提供方**：obra（第三方作者）
- **仓库**：[obra/superpowers](https://github.com/obra/superpowers)
- **组件**：Skills + Hooks
- **作用**：提供头脑风暴、子代理驱动开发（内置代码审查）、系统化调试、red/green TDD 等 skill，以及编写与测试新 skill 的方法

安装：

```bash
claude plugins install superpowers@claude-plugins-official
```

#### `code-review`

- **组件**：Commands
- **作用**：对 PR 做自动代码审查，由多个专职 agent 审查并按置信度过滤误报

安装：

```bash
claude plugins install code-review@claude-plugins-official
```

#### `code-simplifier`

- **组件**：Agents
- **作用**：在功能不变的前提下简化与精炼代码，聚焦最近修改的部分

安装：

```bash
claude plugins install code-simplifier@claude-plugins-official
```

#### `context7`

- **提供方**：Upstash
- **仓库**：[upstash/context7](https://github.com/upstash/context7)
- **组件**：MCP server
- **作用**：将版本匹配的官方文档与代码示例注入上下文；连接远程 MCP，无需本地 Node / npx

安装：

```bash
claude plugins install context7@claude-plugins-official
```

#### `frontend-design`

- **组件**：Skills
- **作用**：生成有设计感、敢于取舍的前端界面，避免模板化的 AI 审美

安装：

```bash
claude plugins install frontend-design@claude-plugins-official
```

#### `skill-creator`

- **组件**：Skills
- **作用**：创建与改进 skill，运行 eval 与方差基准

安装：

```bash
claude plugins install skill-creator@claude-plugins-official
```

#### `mattpocock-skills`

- **提供方**：Matt Pocock
- **仓库**：[mattpocock/skills](https://github.com/mattpocock/skills)
- **组件**：Skills
- **作用**：工程向 skill 合集：追问、spec / 工单流转、TDD、代码审查、领域建模等

安装：

```bash
claude plugins install mattpocock-skills@claude-plugins-official
```

### [anthropics/skills](https://github.com/anthropics/skills)

- **提供方**：Anthropic
- **市场名**：`anthropic-agent-skills`
- **规模**：5 个插件（`document-skills`、`example-skills`、`claude-api`、`academy-guide`、`discernment-nudge`）

添加市场：

```bash
claude plugins marketplace add anthropics/skills
```

#### `document-skills`

- **组件**：Skills
- **作用**：文档处理套件：Excel（xlsx）、Word（docx）、PowerPoint（pptx）、PDF

安装：

```bash
claude plugins install document-skills@anthropic-agent-skills
```

### [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)

- **提供方**：multica-ai
- **市场名**：`karpathy-skills`
- **规模**：仅 `andrej-karpathy-skills` 一个插件

添加市场：

```bash
claude plugins marketplace add multica-ai/andrej-karpathy-skills
```

#### `andrej-karpathy-skills`

- **组件**：Skills
- **作用**：减少 LLM 编码常见错误的行为准则：先想后写、简单优先、外科式改动、目标驱动执行

安装：

```bash
claude plugins install andrej-karpathy-skills@karpathy-skills
```

### [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)

- **提供方**：nextlevelbuilder
- **市场名**：`ui-ux-pro-max-skill`
- **规模**：仅 `ui-ux-pro-max` 一个插件

添加市场：

```bash
claude plugins marketplace add nextlevelbuilder/ui-ux-pro-max-skill
```

#### `ui-ux-pro-max`

- **组件**：Skills
- **作用**：UI/UX 设计知识库：本地可检索的 79 种风格、192 套配色、74 组字体搭配、25 类图表、22 个技术栈指南

安装：

```bash
claude plugins install ui-ux-pro-max@ui-ux-pro-max-skill
```

### [affaan-m/ECC](https://github.com/affaan-m/ECC)

- **提供方**：affaan-m
- **市场名**：`ecc`
- **规模**：仅 `ecc` 一个插件

添加市场：

```bash
claude plugins marketplace add https://github.com/affaan-m/ECC
```

#### `ecc`

- **组件**：Skills + Agents + Hooks + Commands
- **作用**：Agent harness 性能优化系统：68 个 agent、293 个 skill、hooks 与规则，含 AgentShield 安全扫描

安装：

```bash
claude plugins install ecc@ecc
```

### [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc)

- **提供方**：OpenAI
- **市场名**：`openai-codex`
- **规模**：仅 `codex` 一个插件

添加市场：

```bash
claude plugins marketplace add openai/codex-plugin-cc
```

#### `codex`

- **组件**：Skills + Agents + Hooks + Commands
- **作用**：在 Claude Code 内调用 Codex 做代码审查或委派任务（`/codex:review`、`/codex:rescue` 等）；需 ChatGPT 订阅或 OpenAI API key（Node ≥ 18.18）

安装：

```bash
claude plugins install codex@openai-codex
```

> `codex` 安装后需先执行 `/reload-plugins`，再运行一次 `/codex:setup` 完成配置。

---

## Codex

**插件说明**

- **Plugin**：把 skills / MCP servers / browser extensions / hooks 打包成的可安装单元；ChatGPT 与 Codex 共用同一个插件目录
- **Marketplace**：插件来源目录，支持 GitHub 仓库（`owner/repo`，可 `@ref` 钉定版本）、HTTP(S) / SSH Git URL 与本地目录

**常用命令**

```bash
# 市场
codex plugin marketplace add <owner/repo | Git URL | 本地路径>
codex plugin marketplace list
codex plugin marketplace upgrade [name]
codex plugin marketplace remove <name>

# 插件
codex plugin add <plugin>@<marketplace>
codex plugin list [--json]
codex plugin remove <plugin>
```

会话内输入 `/plugins` 打开插件浏览器：按市场分组浏览，`Space` 切换已装插件的启用状态。

**注意事项**

- 安装插件后需**开启新会话**，其捆绑的 skills 与工具才会生效。
- 插件可包含四类组件：Skills（指令）、MCP servers（外部工具连接）、Browser extensions（浏览器能力）、Hooks（生命周期命令）。
- ⚠️ Hooks 会在 Codex 运行周期的设定时机执行命令，启用前应先审查并信任。
- IDE 扩展不支持插件；插件的浏览与安装在 Codex CLI 与 ChatGPT 桌面 App 中进行。
- 以 API key 登录 Codex 时，依赖 OAuth 连接流程的插件不可安装。

市场与插件清单待补充。

## Pi

**插件说明**

- **Package**：Pi 的扩展单元，从 npm、Git 仓库或本地目录安装，可包含 skills、prompts、themes 与扩展工具等资源；Pi 无插件市场概念，按本页层级约定 packages 直接作为二级

**安装层级（scope）**

| scope | 写入的配置 | 生效范围 |
| --- | --- | --- |
| 用户级（默认） | `~/.pi/agent/settings.json` | 本机所有项目 |
| 项目级（`-l`） | `<项目>/.pi/settings.json` | 仅当前项目 |

**常用命令**

```bash
pi install npm:<package>               # 从 npm 安装
pi install git:github.com/user/repo    # 从 Git 仓库安装；亦接受 https://、ssh:// 与本地路径
pi install <source> -l                 # 安装到项目级（.pi/settings.json）
pi list                                # 列出已安装的扩展
pi update [source | self | pi]         # 更新扩展、pi 自身或模型目录
pi remove <source> [-l]                # 移除扩展（uninstall 为别名）
pi config [-l]                         # 打开 TUI 启用 / 禁用包资源（Tab 切换 scope）
```

**注意事项**

- Skill 与 package 相互独立：skills 直接放在 `.agents/skills/`（项目级）与 `~/.agents/skills/`（用户级），无需打包为 package。
- 项目本地（`-l`）文件受信任机制约束，可用 `-a/--approve` 显式信任、`-na/--no-approve` 显式忽略。

packages 清单待补充。
