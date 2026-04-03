---
name: "git-workflow"
description: "自动化 Git 提交、推送和合并工作流。当用户完成代码修改并需要提交到远程仓库并合并到 release 分支时调用此 skill。"
---

# Git 工作流自动化

此 skill 用于自动化执行完整的 Git 提交、推送和合并流程，适用于项目的标准开发流程。

## 使用场景

当用户完成代码修改后，需要：
1. 提交代码更改
2. 推送到远程 dev 分支
3. 合并到对应的 release 分支
4. 推送 release 分支
5. 切换回 dev 分支继续开发

## 前置条件

- 当前在 dev 分支（格式：`dev-x.y.z`）
- 已有未提交的代码更改
- 远程仓库已配置

## 执行步骤

### 1. 检查当前状态 (MANDATORY)
**必须先执行此步骤，确认工作区状态！**
```bash
git status
git branch --show-current
```

### 2. 添加更改文件 (CRITICAL)
**⚠️ 严禁使用 `git add .` 或 `git add -A`！**
**必须显式指定要提交的文件路径，避免误提交编译产物、IDE 配置或自动生成的代码。**

根据用户修改的文件，执行：
```bash
git add <file1> <file2> ...
```

### 3. 提交代码（严格按照项目规范）

**Git Message 格式要求：**
- 格式：`feat <编号>: 描述` 或 `test <编号>: 描述`
- 示例：`feat 180: 修正 strategyPushess 参数命名错误`
- 如果没有特定编号，使用通用的编号（如 180）

**提交命令：**
```bash
git commit -m "feat <编号>: <简短描述>

<详细描述（可选）>"
```

### 4. 推送到远程 dev 分支
```bash
git push
```

### 5. 获取最新远程分支信息
```bash
git fetch origin
```

### 6. 切换到对应的 release 分支

**分支命名规则：**
- dev 分支：`dev-x.y.z`
- release 分支：`release-x.y.z`（版本号相同）

**切换命令：**
```bash
git checkout release-x.y.z
```

### 7. 合并 dev 分支到 release 分支
```bash
git merge dev-x.y.z
```

**处理冲突（如果有）：**
- 如果出现冲突，需要手动解决冲突文件
- 解决后执行：
```bash
git add <冲突文件>
git commit -m "Merge branch 'dev-x.y.z' into release-x.y.z"
```

**无冲突时：**
```bash
git commit -m "Merge branch 'dev-x.y.z' into release-x.y.z"
```

### 8. 推送 release 分支到远程
```bash
git push
```

### 9. 切换回 dev 分支
```bash
git checkout dev-x.y.z
```

### 10. 验证状态
```bash
git status
git log --oneline -3
```

## 重要注意事项

1. **Git Message 必须严格遵循规范**，否则远程仓库会拒绝提交
2. **分支版本号必须对应**：`dev-1.6.7.1` → `release-1.6.7.1`
3. **无需交互确认**，直接执行所有步骤
4. **禁止全量添加**：严禁使用 `git add .`，防止误提交。
5. **提交前检查**：必须确认暂存区不包含无关文件（如 `.class`, `.idea/`, `target/` 等）。
6. 如果合并时出现冲突，需要先解决冲突再继续

## 示例完整流程

假设当前在 `dev-1.6.7.1` 分支，修改了两个文件：

```bash
# 1. 检查状态（必做）
git status

# 2. 添加文件（显式指定）
git add rongan-persona/src/main/java/com/rongan/module/plan/service/PlanStrategyService.java
git add rongan-persona/src/main/java/com/rongan/module/plan/service/impl/PlanStrategyServiceImpl.java

# 3. 提交（严格遵循规范）
git commit -m "feat 180: 修正 strategyPushess 参数命名错误

将参数名从 strategyPushess 修正为 strategyPushes，修复拼写错误。
同时修复了接口定义中多余的空格问题。"

# 4. 推送 dev
git push

# 5. 获取最新信息
git fetch origin

# 6. 切换到 release
git checkout release-1.6.7.1

# 7. 合并
git merge dev-1.6.7.1

# 8. 提交合并
git commit -m "Merge branch 'dev-1.6.7.1' into release-1.6.7.1"

# 9. 推送 release
git push

# 10. 切回 dev
git checkout dev-1.6.7.1
```

## 错误处理

- **推送失败**：检查网络连接和远程仓库权限
- **合并冲突**：手动解决冲突后重新提交
- **分支不存在**：确认 release 分支是否已创建
- **Git Message 格式错误**：按照规范重新提交
