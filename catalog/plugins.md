# AI 插件清单

> 收录各 AI agent 工具的**插件（Plugin）与插件市场（Marketplace）**。结构约定：一级为 AI agent 工具，二级为插件市场，三级为插件；无插件市场概念的工具，下层级依次上提。目前收录 Claude Code，Codex、Pi 待补充。

## Claude Code

Claude Code 的插件体系围绕四个概念展开：

| 概念 | 是什么 |
| --- | --- |
| **Skill** | 一个或多个 `SKILL.md`，Claude 按需加载的指令；可独立放在 `~/.claude/skills/`，不需要插件 |
| **MCP server** | 提供外部系统工具的服务器，声明在 `.mcp.json`；可独立配置，不需要插件 |
| **Plugin** | **打包与分发单元**：把 skills / agents / hooks / MCP servers / commands / LSP / 主题 打成一个可安装单元 |
| **Marketplace** | 含 `.claude-plugin/marketplace.json` 的仓库或目录，**是目录不是商店**，只声明「去哪取」 |

插件可安装到三个层级（scope）：

| scope | 写入的 settings 文件 | 生效范围 |
| --- | --- | --- |
| `user`（`install` 默认） | `~/.claude/settings.json` | 本机所有项目 |
| `project` | `.claude/settings.json`（提交进仓库） | 仓库内所有人 |
| `local` | `.claude/settings.local.json`（gitignore） | 仅你在本仓库 |

- 项目 scope **只提交 `enabledPlugins` 条目，不会替协作者下载插件** —— 每人仍需各自 `claude plugin install <plugin>@<marketplace> --scope project`。
- `managed` 不是安装 scope：组织通过策略文件统一管控，它只在 `claude plugin update --scope` 中可用。
- 同一插件存在于多个 scope 时优先级：`local` > `project` > `user`。
- **落盘目录**：插件根目录 `~/.claude/plugins`（可用 `CLAUDE_CODE_PLUGIN_CACHE_DIR` 改），下含 `cache/<marketplace>/<plugin>/<version>/`、`marketplaces/<name>/`、`data/<plugin-id>/`、`installed_plugins.json`、`known_marketplaces.json`。

以下按插件市场列出本清单收录的插件。

### `claude-plugins-official`（官方内置市场）

Anthropic 官方市场，仓库为 `anthropics/claude-plugins-official`，**首次启动交互式会话时自动添加**，常规使用无需手动 `add`。内容规模：**315 个插件条目** = Anthropic 自研 39 个（含 15 个语言服务器插件）+ 合作方内嵌 14 个 + 远程引用 262 个；**默认开启自动更新**（其他第三方市场默认关闭）。

仅在「从没有人开过交互式会话」的机器上（如 CI 脚本）需要显式添加：

```bash
claude plugin marketplace add anthropics/claude-plugins-official
```

#### `superpowers`

**提供方**：obra（第三方作者）
**作用**：教 Claude 头脑风暴、子代理驱动开发（内置代码审查）、系统化调试、red/green TDD，以及如何编写与测试新 skill
**安装**：

```bash
claude plugins install superpowers@claude-plugins-official
```

#### `code-review`

**提供方**：Anthropic
**作用**：PR 自动代码审查：多个专职 agent + 基于置信度的打分以过滤误报（Commands）
**安装**：

```bash
claude plugins install code-review@claude-plugins-official
```

#### `code-simplifier`

**提供方**：Anthropic
**作用**：在保持功能不变的前提下简化与精炼代码，聚焦最近修改的部分（Agents）
**安装**：

```bash
claude plugins install code-simplifier@claude-plugins-official
```

#### `context7`

**提供方**：Upstash
**作用**：Context7 MCP：拉取版本相关的官方文档与代码示例进上下文，连接远程 MCP，**无需本地 Node / npx**（可匿名使用，设 `CONTEXT7_API_KEY` 提高限流）
**安装**：

```bash
claude plugins install context7@claude-plugins-official
```

#### `frontend-design`

**提供方**：Anthropic
**作用**：生成有设计感、敢做取舍的前端界面，避免通用的 AI 审美（Skills）
**安装**：

```bash
claude plugins install frontend-design@claude-plugins-official
```

#### `skill-creator`

**提供方**：Anthropic
**作用**：创建新 skill、改进已有 skill、跑 eval 与方差基准（Skills）
**安装**：

```bash
claude plugins install skill-creator@claude-plugins-official
```

#### `mattpocock-skills`

**提供方**：Matt Pocock
**作用**：工程向 skills 合集：追问、spec / 工单流转、TDD、代码审查、领域建模等
**安装**：

```bash
claude plugins install mattpocock-skills@claude-plugins-official
```

### `anthropic-agent-skills`（anthropics/skills）

Anthropic 官方 skills 仓库提供的市场，含 5 个插件；本清单安装 `document-skills`，其余（`example-skills`、`claude-api`、`academy-guide`、`discernment-nudge`）按需安装。

添加市场（仅需执行一次）：

```bash
claude plugins marketplace add anthropics/skills
```

#### `document-skills`

**提供方**：Anthropic
**作用**：文档处理套件：Excel（xlsx）、Word（docx）、PowerPoint（pptx）、PDF
**安装**：

```bash
claude plugins install document-skills@anthropic-agent-skills
```

### `karpathy-skills`（multica-ai/andrej-karpathy-skills）

该市场仅提供 `andrej-karpathy-skills` 一个插件。

添加市场（仅需执行一次）：

```bash
claude plugins marketplace add multica-ai/andrej-karpathy-skills
```

#### `andrej-karpathy-skills`

**提供方**：multica-ai
**作用**：减少 LLM 编码常见错误的行为准则：先想后写、简单优先、外科式改动、目标驱动执行
**安装**：

```bash
claude plugins install andrej-karpathy-skills@karpathy-skills
```

### `ui-ux-pro-max-skill`（nextlevelbuilder/ui-ux-pro-max-skill）

该市场仅提供 `ui-ux-pro-max` 一个插件。

添加市场（仅需执行一次）：

```bash
claude plugins marketplace add nextlevelbuilder/ui-ux-pro-max-skill
```

#### `ui-ux-pro-max`

**提供方**：nextlevelbuilder
**作用**：UI/UX 设计智能：本地可检索的 79 种风格、192 套配色、74 组字体搭配、25 类图表、22 个技术栈指南
**安装**：

```bash
claude plugins install ui-ux-pro-max@ui-ux-pro-max-skill
```

### `ecc`（affaan-m/ECC）

该市场仅提供 `ecc` 一个插件。

添加市场（仅需执行一次）：

```bash
claude plugins marketplace add https://github.com/affaan-m/ECC
```

#### `ecc`

**提供方**：affaan-m
**作用**：Agent harness 性能优化系统：68 个 agent、293 个 skill、hooks 与规则，含 AgentShield 安全扫描
**安装**：

```bash
claude plugins install ecc@ecc
```

### `openai-codex`（openai/codex-plugin-cc）

该市场仅提供 `codex` 一个插件。

添加市场（仅需执行一次）：

```bash
claude plugins marketplace add openai/codex-plugin-cc
```

#### `codex`

**提供方**：OpenAI
**作用**：在 Claude Code 内调用 Codex 做代码审查或把任务委派给它（`/codex:review`、`/codex:adversarial-review`、`/codex:rescue`、`/codex:transfer` 等）；需 ChatGPT 订阅或 OpenAI API key，Node ≥ 18.18
**安装**：

```bash
claude plugins install codex@openai-codex
```

> `codex` 装完需先执行 `/reload-plugins`，再跑一次 `/codex:setup` 完成配置。

**常用命令**

```bash
# 市场
claude plugin marketplace add <owner/repo | URL | 本地路径>
claude plugin marketplace list
claude plugin marketplace update [name]        # 只刷新市场目录，不更新已装插件版本
claude plugin marketplace remove <name>        # 会连带卸载从该市场装的插件

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

一次性「加市场 + 装插件」（需 v2.1.275+）：

```bash
/plugin install <plugin> --marketplace <source>
```

**注意事项**

- ⚠️ **插件是有成本的**：启用后，其每个 skill / agent / command 的**名称与描述会进入每一轮上下文**（正文只在真正使用时加载）。`claude plugin details <name>` 可查看 `Always-on` token 数与逐组件成本。
- ⚠️ 市场名 `claude-plugins-official`、`anthropic-agent-skills` 等**官方名被保留**，第三方冒用会报 `The name '<name>' is reserved for official Anthropic marketplaces`。
- 官方**不维护「市场内插件清单」文档页**（"The catalog changes often, so this page doesn't list it"）。查最新清单用会话内 `/plugin` 的 **Discover** 标签，或网页版 <https://claude.com/marketplace/plugins>。
- `marketplace remove` 用 **`marketplace.json` 里的 `name` 字段**，不是当初传给 `add` 的 source（两者可能不同，例如 `anthropics/skills` 的市场名是 `anthropic-agent-skills`）。
- 卸载时若该插件仍被另一个已启用插件依赖，会被拒绝并给出链式命令。
- 从**本地路径**添加的市场中的插件是就地加载，改源码下次会话即生效，**不需要升版本号**；其他市场插件会复制到 `~/.claude/plugins/cache/` 后加载。
- 云会话（含浏览器里的 claude.ai/code）**不加载**本地设置中的插件。

---

## Codex

> 待补充。已知命令：`codex plugin marketplace add <owner/repo>`、`codex plugin add <plugin>@<marketplace>`。

## Pi

> 待补充。已知：Pi 的扩展形态是 **packages**（`pi install npm:<package>`）而非 marketplace——无市场概念，按本页层级约定 packages 直接作为二级；skills 走 `.agents/skills/`（项目级）与 `~/.agents/skills/`（用户级）。
