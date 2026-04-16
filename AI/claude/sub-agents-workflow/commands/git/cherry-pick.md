---
description: 将指定提交 cherry-pick 到新分支，并推送创建 MR。若 API 失败则输出手动创建链接。
---

## 执行逻辑

### 1. 输入参数
调用方需传入：
- `COMMIT_SHA`：要 cherry-pick 的提交哈希（必填）
- `TARGET_BRANCH`：目标分支，即 MR 要合入的分支（必填）
- `NEW_BRANCH_NAME`：新分支名（可选，默认自动生成如 `cherry-pick-<COMMIT_SHA短哈希>`）
- `MR_TITLE`：MR 标题（可选，默认使用原提交标题）
- `MR_DESCRIPTION`：MR 描述（可选）
- `REVIEWER_IDS`：审核人数字 ID 数组（可选）

### 2. 前置检查
- 确保当前工作区干净（无未提交变更），否则通过 `/git:stash` 暂存。
- 确保 `.claude/settings.json` 中已配置 GitLab 相关字段。

### 3. 执行步骤

#### 3.1 创建并切换到新分支
```bash
/git:branch <NEW_BRANCH_NAME>
```

#### 3.2 执行 cherry-pick
```bash
git cherry-pick <COMMIT_SHA>
```
若发生冲突，立即停止并提示：
```
❌ CHERRY_PICK_CONFLICT: 请手动解决冲突后继续，或放弃操作。
```
指令退出，不进行后续步骤。

#### 3.3 推送新分支
```bash
/git:push origin HEAD
```

#### 3.4 生成 MR 标题（若未提供）

- 若原提交已含 `feat:` / `fix:` 等前缀，直接使用。
- 否则添加 `chore(cherry-pick):` 前缀。

#### 3.5 生成 MR 描述（若未提供）

- 新增中文描述模板，包含原提交信息和变更摘要，保护ONES相关信息。

#### 3.6 创建 MR

调用 `/git:merge` 的逻辑（可复用或内联），传入：
- `source_branch`: 新分支名
- `target_branch`: 目标分支
- `title`: MR 标题
- `description`: MR 描述
- `reviewer_ids`: 审核人 ID

#### 3.7 降级处理
若 MR 创建失败（API 返回非 2xx），输出手动创建链接，格式如下：
```
⚠️ 自动创建 MR 失败，请手动创建：
🔗 <GITLAB_HOST>/<项目路径>/-/merge_requests/new?merge_request%5Bsource_branch%5D=<新分支名>&merge_request%5Btarget_branch%5D=<目标分支>
```

### 4. 清理
成功或失败后，建议切回原分支（若需要），但非强制。

## 依赖
- `/git:branch`、`/git:push`、`/git:merge` 原子指令
- `git` 命令
- `curl`、`jq`（用于 MR 创建）

## 错误码
- `0`：成功
- `1`：参数缺失或 cherry-pick 冲突
- `2`：MR 创建失败（已降级输出链接）
## 二、使用方式

### 用户直接调用

```txt
/git:cherry-pick COMMIT_SHA=abc123 TARGET_BRANCH=release/1.0
```

### Orchestrator 自动调用
当用户说：“把提交 abc123 合入 release/1.0 分支”时，Orchestrator 识别意图后调度 `implementer`，`implementer` 调用 `/git:cherry-pick` 完成操作。