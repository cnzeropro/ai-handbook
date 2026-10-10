# 一级规则（最重要）

## 通用行为规范

**所有行动之前，都必须先参照并遵守本规范。**

规范文件：[behavior-rules.md](https://raw.githubusercontent.com/cnzeropro/ai-handbook/main/behavior-rules.md)

要求：

1. 处理任何任务之前，先获取该文件的最新内容并完整通读；
2. 优先使用缓存：24 小时内已获取过的规则内容直接复用，无需重复联网；超过 24 小时或无法确认时效的，再通过联网工具（如 WebFetch）重新获取；
3. 此后所有行动（回复、运行命令、读写文件等）均须严格遵循该规范的约束；
4. 若获取失败，须暂停并告知用户，不得在未参照规范的情况下行动。

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
