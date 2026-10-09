# AI Skills 清单

> 通过 [`npx skills`](https://github.com/vercel-labs/skills)（vercel-labs 官方 CLI，latest 1.7.0，需 Node ≥ 22.20）安装外部 Agent Skills。
> 不带 `-g` 时安装到当前项目（`.claude/skills/`，随仓库提交、团队共享）；带 `-g` 时安装到用户级（`~/.claude/skills/`，所有项目可用）。
> skill 的**名称与描述会进入每个会话的上下文**，安装数量越多，每轮上下文开销越大。

## 分层判据

| 层级 | 判据 |
| --- | --- |
| **全局（`-g`）** | 元技能或工具型能力，与具体代码库无关 |
| **项目级** | 绑定具体代码库 / 技术栈，或只有部分项目需要 |

## 常用命令

```bash
npx skills add <owner/repo>              # 安装到当前项目
npx skills add <owner/repo> -g           # 安装到用户级（跨项目）
npx skills add <owner/repo> -l           # 仅列出仓库包含的 skill，不安装
npx skills list [-g]                     # 查看已安装的 skill（默认仅项目级）
npx skills update [-g]                   # 更新
npx skills remove <skill> [-g]           # 卸载
npx skills find [关键词]                  # 搜索技能
```

其他参数：`-a/--agent <agents>` 指定目标客户端（claude-code / codex / cursor / pi 等，支持 80+）；`-y/--yes` 跳过确认；`--copy` 用复制代替软链接；`--all` 等价 `--skill '*' --agent '*' -y`。

## 注意事项

- ⚠️ **不带 `--skill` 时会全部安装**：仓库仅含 1 个 skill 时、或传入 `-y` 时选择全部；在 AI agent 会话中执行会自动等同于 `-y`（CLI 检测到 agent 运行环境即进入非交互模式）。多 skill 的仓库建议显式列出 `--skill`。

## Skills 一览

以下按仓库热度列出本清单收录的 skills。

### [mattpocock/skills](https://github.com/mattpocock/skills)

- **规模**：38 个 skill

> 该仓库同时以官方插件形式发布（`mattpocock-skills`），插件与 `npx skills` 两条安装路线二选一，同时安装会造成 skill 重复。

#### `tdd`

**作用**：测试驱动开发（red-green-refactor、集成测试）

**安装**：

```bash
npx skills add mattpocock/skills --skill tdd
```

#### `grill-me`

**作用**：针对方案或设计持续追问，帮助完善方案

**安装**：

```bash
npx skills add mattpocock/skills --skill grill-me
```

#### `grill-with-docs`

**作用**：与 `grill-me` 相同，并产出文档（ADR 与术语表）

**安装**：

```bash
npx skills add mattpocock/skills --skill grill-with-docs
```

#### `improve-codebase-architecture`

**作用**：扫描代码库寻找「深化」机会 → 输出可视化 HTML 报告 → 选择一项继续深入追问

**安装**：

```bash
npx skills add mattpocock/skills --skill improve-codebase-architecture
```

### [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)

- **规模**：1 个 skill

#### `karpathy-guidelines`

**作用**：减少 LLM 编码通病的四条准则：先想后写 / 简单优先 / 外科式改动 / 目标驱动执行

**安装**：

```bash
npx skills add multica-ai/andrej-karpathy-skills -g
```

### [anthropics/skills](https://github.com/anthropics/skills)

- **规模**：20 个 skill

#### `skill-creator`

**作用**：创建新 skill、改进已有 skill、运行 eval 与方差基准、优化触发描述

**安装**：

```bash
npx skills add anthropics/skills --skill skill-creator -g
```

#### `mcp-builder`

**作用**：构建高质量 MCP server（Python FastMCP 或 Node / TypeScript SDK）

**安装**：

```bash
npx skills add anthropics/skills --skill mcp-builder -g
```

#### `xlsx`

**作用**：电子表格读写与清洗（.xlsx / .xlsm / .csv），公式、格式化、图表

**安装**：

```bash
npx skills add anthropics/skills --skill xlsx -g
```

#### `docx`

**作用**：创建 / 编辑 Word（.docx），含目录、页码、页眉、修订、批注、查找替换

**安装**：

```bash
npx skills add anthropics/skills --skill docx -g
```

#### `pptx`

**作用**：创建 / 解析 / 编辑 PowerPoint（.pptx），模板、版式、演讲者备注

**安装**：

```bash
npx skills add anthropics/skills --skill pptx -g
```

#### `pdf`

**作用**：PDF 读取、合并拆分、旋转、水印、表单填写、加解密、抽图、OCR

**安装**：

```bash
npx skills add anthropics/skills --skill pdf -g
```

#### `frontend-design`

**作用**：新建或重塑 UI 时的视觉设计指导，避免模板化的 AI 味

**安装**：

```bash
npx skills add anthropics/skills --skill frontend-design
```

### [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)

- **规模**：7 个 skill

#### `ui-ux-pro-max`

**作用**：UI/UX 设计知识库：本地可检索的 79 种风格、192 套配色、74 组字体搭配、25 类图表、22 个技术栈指南

**安装**：

```bash
npx skills add nextlevelbuilder/ui-ux-pro-max-skill --skill ui-ux-pro-max
```

### [stablyai/orca](https://github.com/stablyai/orca)

- **规模**：8 个 skill

#### `orca-cli`

**作用**：用 `orca` CLI 操作 worktree、文件夹上下文、终端、仓库、自动化、artifact、内嵌浏览器

**安装**：

```bash
npx skills add stablyai/orca --skill orca-cli -g
```

#### `computer-use`

**作用**：通过 `orca computer` 驱动本地可见窗口 GUI（无障碍树、点击、输入、菜单、截图）

**安装**：

```bash
npx skills add stablyai/orca --skill computer-use -g
```

#### `orchestration`

**作用**：协调受监督的 Orca worker：线程消息、阻塞式 ask/reply、任务派发、任务 DAG、决策门、coordinator 循环

**安装**：

```bash
npx skills add stablyai/orca --skill orchestration -g
```

### [vercel-labs/skills](https://github.com/vercel-labs/skills)

- **规模**：1 个 skill；该仓库为 `npx skills` CLI 本身

#### `find-skills`

**作用**：用户问「怎么做 X」「有没有现成技能」时，帮助发现并安装相应 skill

**安装**：

```bash
npx skills add vercel-labs/skills -g
```

### [microsoft/playwright-cli](https://github.com/microsoft/playwright-cli)

- **规模**：2 个 skill

#### `playwright-cli`

**作用**：浏览器自动化、测试 Web 页面、编写 Playwright 测试

**安装**：

```bash
npx skills add microsoft/playwright-cli --skill playwright-cli
```

### [huggingface/skills](https://github.com/huggingface/skills)

- **规模**：25 个 skill

#### `hf-cli`

**作用**：Hugging Face Hub CLI：下载 / 上传模型、数据集、Spaces、buckets、repos、papers、jobs

**安装**：

```bash
npx skills add huggingface/skills --skill hf-cli -g
```
