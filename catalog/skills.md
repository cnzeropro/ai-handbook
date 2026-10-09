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

- 官方层级口径：
  - `stablyai/orca` —— 默认全局（原文 "Default scope is **global** (`--global`); pass `--local` for the current project only."）
  - `mattpocock/skills` —— "once per repo"
  - `ui-ux-pro-max` —— 推荐项目级（global 为可选项）
  - `playwright-cli` —— 安装到项目目录
- ⚠️ **不带 `--skill` 时会全部安装**：仓库仅含 1 个 skill 时、或传入 `-y` 时选择全部；**在 AI agent 会话中执行会自动等同于 `-y`**（CLI 检测到运行环境是 agent 即进入非交互模式）。多 skill 的仓库建议显式列出 `--skill`。
- `mattpocock/skills` 官方提醒：**插件路线与 `npx skills` 路线二选一**，"installing both leaves you with every skill twice."

## Skills 一览

以下按仓库热度列出本清单收录的 skills。

#### [mattpocock/skills](https://github.com/mattpocock/skills)

- **规模**：本清单安装 4 个 skill

安装：

```bash
npx skills add mattpocock/skills --skill grill-me --skill grill-with-docs --skill improve-codebase-architecture --skill tdd
```

- **`tdd`**：测试驱动开发（red-green-refactor、集成测试）
- **`grill-me`**：针对方案或设计持续追问，帮助完善方案
- **`grill-with-docs`**：与 `grill-me` 相同，并产出文档（ADR 与术语表）
- **`improve-codebase-architecture`**：扫描代码库寻找「深化」机会 → 输出可视化 HTML 报告 → 选择一项继续深入追问

#### [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)

- **规模**：仅 `karpathy-guidelines` 一个 skill

安装：

```bash
npx skills add multica-ai/andrej-karpathy-skills -g
```

- **`karpathy-guidelines`**：减少 LLM 编码通病的四条准则：先想后写 / 简单优先 / 外科式改动 / 目标驱动执行

#### [anthropics/skills](https://github.com/anthropics/skills)

- **规模**：19 个 skill，本清单安装 7 个（全局 6 个、项目级 1 个），其余（`webapp-testing`、`web-artifacts-builder`、`theme-factory`、`canvas-design` 等）可按需安装

安装：

```bash
npx skills add anthropics/skills --skill skill-creator --skill mcp-builder -g
npx skills add anthropics/skills --skill xlsx --skill docx --skill pdf --skill pptx -g
npx skills add anthropics/skills --skill frontend-design
```

- **`skill-creator`**：创建新 skill、改进已有 skill、运行 eval 与方差基准、优化触发描述
- **`mcp-builder`**：构建高质量 MCP server（Python FastMCP 或 Node / TypeScript SDK）
- **`xlsx`**：电子表格读写与清洗（.xlsx / .xlsm / .csv），公式、格式化、图表
- **`docx`**：创建 / 编辑 Word（.docx），含目录、页码、页眉、修订、批注、查找替换
- **`pptx`**：创建 / 解析 / 编辑 PowerPoint（.pptx），模板、版式、演讲者备注
- **`pdf`**：PDF 读取、合并拆分、旋转、水印、表单填写、加解密、抽图、OCR
- **`frontend-design`**：新建或重塑 UI 时的视觉设计指导，避免模板化的 AI 味

#### [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)

- **规模**：仅 `ui-ux-pro-max` 一个 skill

安装：

```bash
npx skills add nextlevelbuilder/ui-ux-pro-max-skill --skill ui-ux-pro-max
```

- **`ui-ux-pro-max`**：UI/UX 设计知识库：本地可检索的 79 种风格、192 套配色、74 组字体搭配、25 类图表、22 个技术栈指南

#### [stablyai/orca](https://github.com/stablyai/orca)

- **规模**：本清单安装 3 个 skill

安装：

```bash
npx skills add stablyai/orca --skill orca-cli --skill computer-use --skill orchestration -g
```

- **`orca-cli`**：用 `orca` CLI 操作 worktree、文件夹上下文、终端、仓库、自动化、artifact、内嵌浏览器
- **`computer-use`**：通过 `orca computer` 驱动本地可见窗口 GUI（无障碍树、点击、输入、菜单、截图）
- **`orchestration`**：协调受监督的 Orca worker：线程消息、阻塞式 ask/reply、任务派发、任务 DAG、决策门、coordinator 循环

#### [vercel-labs/skills](https://github.com/vercel-labs/skills)

- **规模**：`npx skills` CLI 本身的仓库，含 2 个 skill（`find-skills`、`pr-labeling`），本清单安装 `find-skills`

安装：

```bash
npx skills add vercel-labs/skills -g
```

- **`find-skills`**：用户问「怎么做 X」「有没有现成技能」时，帮助发现并安装相应 skill

#### [microsoft/playwright-cli](https://github.com/microsoft/playwright-cli)

- **规模**：本清单安装 `playwright-cli` 一个 skill

安装：

```bash
npx skills add microsoft/playwright-cli --skill playwright-cli
```

- **`playwright-cli`**：浏览器自动化、测试 Web 页面、编写 Playwright 测试

#### [huggingface/skills](https://github.com/huggingface/skills)

- **规模**：25 个 skill，本清单仅安装入口 `hf-cli`，其余（`huggingface-datasets`、`huggingface-spaces`、`transformers-js`、`trl-training` 等）可按需安装

安装：

```bash
npx skills add huggingface/skills --skill hf-cli -g
```

- **`hf-cli`**：Hugging Face Hub CLI：下载 / 上传模型、数据集、Spaces、buckets、repos、papers、jobs
