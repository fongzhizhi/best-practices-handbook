# Git 工作流指令使用指南

本指南帮助你快速上手基于 Claude Code 的 Git 自动化指令集。这些指令既可**独立使用**，也能**智能协作**，覆盖从切换分支到创建 MR 的完整开发流程。

## 指令速览

| 指令               | 用途                       | 常用场景                         |
| :----------------- | :------------------------- | :------------------------------- |
| `/git-checkout`    | 切换分支并拉取最新代码     | 开始任务前切换到基准分支         |
| `/git-branch`      | 智能创建符合规范的分支     | 开始新功能或为已完成代码创建分支 |
| `/prettier-format` | 格式化变更文件             | 提交前统一代码风格               |
| `/git-commit`      | 生成规范 commit 并提交     | 提交代码，自动关联 ONES 和 Spec  |
| `/git-stash`       | 暂存未提交变更             | 临时切换分支或保存现场           |
| `/git-push`        | 推送代码并自动处理前置步骤 | 将提交推送到远程                 |
| `/git-merge`       | 创建 GitLab MR             | 提交代码审查，完成开发闭环       |

## 典型工作流

### 工作流一：先建分支，再开发（推荐）

```text
1. /git-checkout develop          # 切换到开发基准分支
2. /git-branch                    # 创建功能分支（智能命名）
3. [进行开发...]
4. /git-push                      # 自动格式化 → 提交 → 推送
5. /git-merge                     # 创建 MR，指定目标分支和审核人
```

### 工作流二：先开发，后建分支

```text
1. [在本地直接开发...]
2. /git-branch                    # 创建分支并自动迁移已写代码
3. /git-push                      # 自动格式化 → 提交 → 推送
4. /git-merge                     # 创建 MR
```

## 各指令详解

### 1. `/git-checkout` —— 切换分支

**作用**：切换到指定分支并拉取最新代码。

**智能特性**：
- 若当前有未提交变更，提供选项：自动暂存后切换 / 仅暂存不恢复 / 手动处理。
- 若目标分支在远程存在但本地没有，自动基于远程创建并设置跟踪。
- 切换后自动执行 `git pull`，冲突时提供变基/合并选项。

**用法**：
```bash
/git-checkout main               # 切换到 main 分支
/git-checkout                    # 交互式选择或输入分支名
```

---

### 2. `/git-branch` —— 创建分支

**作用**：智能创建符合团队命名规范的分支。

**智能特性**：
- **场景 A**（有未提交代码）：自动暂存变更 → 创建分支 → 恢复变更。
- **场景 B**（工作区干净）：直接创建新分支。
- 自动从对话上下文提取 ONES 单信息，生成分支名（格式：`分支来源/类型/ONES_ID`）。
- 分支名冲突时提供：切换已有 / 删除重建 / 手动处理。
- 支持通过 `--base` 指定基准分支（默认为当前分支）。

**用法**：
```bash
/git-branch                      # 智能创建分支
/git-branch --base develop       # 基于 develop 创建
/git-branch feat/avatar-upload   # 手动指定分支名
```

**分支名规范**：
```
<分支来源>/<变更类型>/<ONES_ID或描述>
```
- 分支来源：如 `main`、`develop`、`3.2`
- 变更类型：`feat` / `fix` / `refactor` / `docs` / `style` / `test` / `chore` / `perf` / `ci`
- ONES_ID或描述：关联单号或简短英文描述

---

### 3. `/prettier-format` —— 格式化代码

**作用**：对本次变更的文件执行 Prettier 格式化。

**智能特性**：
- 自动检测 `package.json` 中的 `format` 脚本并优先使用。
- 若无配置，自动使用 `npx prettier` 处理 `.ts/.tsx/.js/.jsx/.json/.md` 文件。
- 仅格式化 `git diff` 中变更的文件，避免触碰无关代码。
- 若项目未配置格式化脚本，会提示可复制的配置内容。

**用法**：
```bash
/prettier-format                 # 格式化所有变更文件
```

---

### 4. `/git-commit` —— 生成并提交

**作用**：生成符合 Conventional Commits 规范的 commit message，并执行提交。

**智能特性**：
- 若暂存区为空，自动暂存所有变更。
- 从对话上下文提取 ONES 单信息（ID、标题、链接）和 Spec 文档路径。
- 生成的 commit 包含 `ONES:`、`Spec:`、`Link:` 尾部标记。
- 提交前展示完整预览，支持确认/修改/取消。

**用法**：
```bash
/git-commit                      # 自动生成并提交
/git-commit --ones PRO-151081    # 手动指定 ONES 单号
```

**Commit 格式示例**：

```
fix(editor): 修复PCB打开时的权限校验错误

- 将权限检查前置到文件读取之前
- 增加无权限时的友好提示

ONES: PRO-151081 【编辑器】编辑器打开PCB报错
Spec: openspec/specs/editor/permission/spec.md
Link: http://ones.lceda/xxxx
```

---

### 5. `/git-stash` —— 暂存变更

**作用**：将当前未提交变更保存到 Git Stash。

**智能特性**：
- 自动分析变更内容，生成有意义的消息（如 `fix(editor): 修复权限校验`）。
- 支持用户确认或自定义消息。
- 默认包含未跟踪文件（`-u`）。

**用法**：
```bash
/git-stash                       # 智能暂存
/git-stash --message "临时保存"  # 自定义消息
```

---

### 6. `/git-push` —— 推送代码

**作用**：将当前分支推送到远程，自动处理格式化、提交等前置步骤。

**智能特性**：
- **未提交变更**：提供选项自动调用 `/prettier-format` + `/git-commit`，或暂存后继续。
- **主分支保护**：若在 `main/master/develop` 上推送，提供创建功能分支或强制推送选项。
- **远程分支不存在**：自动创建并设置上游（`-u`）。
- **推送冲突**：提供变基/合并/强制推送选项。
- 推送后若曾暂存变更，询问是否恢复。

**用法**：
```bash
/git-push                        # 智能推送
/git-push --skip-format          # 跳过格式化步骤
```

---

### 7. `/git-merge` —— 创建 MR

**作用**：通过 Git Push Options 在 GitLab 上创建合并请求（无需安装 `glab`）。

**智能特性**：
- **未提交变更**：自动调用 `/git-push` 完成提交推送。
- **未推送提交**：提示并自动推送。
- 自动确定目标分支（用户指定 → 配置文件 → 仓库默认分支 → 询问）。
- 自动获取审核人（用户指定 → 配置文件 → CODEOWNERS → 询问）。
- 生成 MR 标题和描述（复用 commit 信息及 ONES/Spec 关联）。

**用法**：
```bash
/git-merge                       # 创建 MR（使用默认目标分支）
/git-merge --target develop      # 指定目标分支
/git-merge --reviewer @alice     # 指定审核人
/git-merge -t main -r @alice     # 同时指定
```

**前置要求**：GitLab ≥ 11.10，Git ≥ 2.10（推荐 2.18+）。

---

## 快捷组合示例

### 场景：紧急修复一个 Bug
```bash
/git-checkout main               # 1. 切换到主分支
/git-branch                      # 2. 创建 fix 分支（自动识别 ONES）
# ... 修复代码 ...
/git-push                        # 3. 一键格式化 → 提交 → 推送
/git-merge -t main -r @tech-lead # 4. 创建 MR 到 main，指定审核人
```

### 场景：功能开发到一半需要切换分支
```bash
/git-stash                       # 1. 暂存当前进度
/git-checkout develop            # 2. 切换到其他分支处理紧急事务
# ... 处理完毕 ...
/git-checkout feat/avatar        # 3. 切回原分支
git stash pop                    # 4. 手动恢复（或让 /git-checkout 自动恢复）
```

---

## 常见问题

### Q1：指令执行时提示找不到 `xxx` 指令怎么办？
A：确保所有指令文件已存放在 `.claude/commands/` 目录下，文件名与指令名一致（如 `git-push.md`）。Claude Code 会自动加载。

### Q2：为什么 `/git-merge` 创建 MR 失败？
A：检查 GitLab 版本是否 ≥ 11.10，以及当前分支是否已推送到远程。可尝试先手动执行 `git push` 再运行 `/git-merge`。

### Q3：不想每次都输入审核人，如何配置默认值？
A：在项目根目录创建 `.claude/config.json`，添加：
```json
{
  "mr": {
    "defaultTargetBranch": "develop",
    "defaultReviewers": ["@alice", "@bob"]
  }
}
```

### Q4：分支名冲突时选择了“切换已有分支”，但之前写的代码不见了？
A：代码已通过 `git stash` 暂存。切换到已有分支后，指令会询问是否恢复暂存，选择“是”即可恢复。也可手动执行 `git stash pop`。

### Q5：`/git-push` 推送时提示冲突怎么办？
A：指令会提供变基（rebase）、合并（merge）、强制推送三个选项。建议优先选择“变基后推送”，保持历史线性。

---

## 联系与反馈

如有问题或改进建议，请在本仓库提交 Issue 或联系架构组。