请将当前分支的提交推送到远程仓库，并通过 Git Push Options 在 GitLab 上创建合并请求（Merge Request）。本指令可独立使用，无需安装额外 CLI 工具，智能处理未提交代码、目标分支指定、审核人获取等环节。

## 执行步骤

### 1. 环境检查
- 确认 Git 版本支持 Push Options：执行 `git version` 检查，建议 Git 2.18+。
- GitLab 版本需 ≥ 11.10（通常企业版均满足）。

### 2. 状态检测
- 执行 `git status --porcelain` 检查工作区和暂存区状态。
- 执行 `git branch --show-current` 获取当前分支名。
- 执行 `git rev-parse --abbrev-ref @{upstream} 2>/dev/null` 检查上游跟踪分支。
- 执行 `git log --oneline origin/<当前分支名>..HEAD 2>/dev/null` 检查是否有未推送的本地提交。

### 3. 未提交变更处理
- 若存在未提交变更（工作区或暂存区非空），提示用户：

  ```txt
  ⚠️ 当前有未提交的变更，创建 MR 前需先提交并推送。
  请选择处理方式：
  A. 自动提交并推送（调用 /git-push）
  B. 暂存变更并继续（调用 /git-stash）
  C. 取消操作，手动处理
  ```

    - 若选 `A`：调用 `/git-push` 完成提交和推送，然后继续后续步骤。

    - 若选 `B`：调用 `/git-stash` 暂存变更，然后继续后续步骤。

    - 若选 `C`：终止流程。


### 4. 未推送提交处理
- 若存在未推送的本地提交（或未设置上游分支），提示用户：

  ```txt
  ℹ️ 当前分支有未推送的提交，创建 MR 前需先推送到远程。
  是否立即推送？(y/n)
  ```

    - 若选 `y`：调用 `/git-push` 完成推送。

    - 若选 `n`：终止流程。


### 5. 确定目标分支
按以下优先级确定 MR 的目标分支：
1. 用户通过 `--target` 或 `-t` 参数指定的分支。
2. 项目配置文件 `.claude/config.json` 中的 `mr.defaultTargetBranch` 字段。
4. 若以上均无法确定，询问用户输入目标分支名。

### 6. 确定审核人（Assignee）
按以下优先级获取审核人（GitLab 用户名，多个用逗号分隔）：
1. 用户通过 `--reviewer` 或 `-r` 参数指定的审核人。
2. 项目配置文件 `.claude/config.json` 中的 `mr.defaultReviewers` 数组。
4. 若以上均无法获取，询问用户输入审核人（可回车跳过）。

### 7. 生成 MR 标题和描述
- **标题**：优先使用最近一次 commit message 的标题行（`<type>(<scope>): <简短描述>`）。若无法获取，则根据分支名和变更内容智能生成。

- **描述**：包含以下部分（若有则填写）：

  ```txt
  ## 变更说明
  
  <最近一次 commit 的正文>
  
  ## 关联信息
  
  - ONES: <ONES单ID> <ONES单标题>
  - Spec: <Spec文档路径>
  - Link: <ONES单URL>
  ```

  其中 ONES 和 Spec 信息从当前对话上下文或最近 commit 中提取。

### 8. 构造 Git Push 命令
基于收集的信息，构建带 Push Options 的推送命令。

**基础推送命令**（若已设置上游）：
```bash
git push
```

**完整命令示例**（首次推送或需附加选项）：
```bash
git push -u origin HEAD \
  -o merge_request.create \
  -o merge_request.target=<目标分支> \
  -o merge_request.title="<标题>" \
  -o merge_request.description="<描述>" \
  -o merge_request.assign="<审核人>" \
  -o merge_request.remove_source_branch
```

选项说明：
- `-u origin HEAD`：推送当前分支并设置上游（若未设置）。
- `-o merge_request.create`：触发创建 MR。
- `-o merge_request.target=<分支>`：指定目标分支。
- `-o merge_request.title="..."`：设置 MR 标题。
- `-o merge_request.description="..."`：设置 MR 描述（注意描述中的换行和引号需转义）。
- `-o merge_request.assign="用户名"`：指定分配人（审核人）。
- `-o merge_request.remove_source_branch`：合并后自动删除源分支。

### 9. 执行推送与 MR 创建
- 若用户选择暂存了变更（步骤 3 选 B），先执行 `git stash pop` 恢复（可选询问）。
- 执行构造的 `git push` 命令。
- 解析命令输出，提取 GitLab 返回的 MR 链接（通常格式为 `remote: https://gitlab.example.com/...`）。
- 若推送成功且 MR 已创建，输出结果。

### 10. 输出结果与后续提示
```markdown
✅ 推送成功，合并请求已创建
链接：<MR 的 Web URL>

目标分支：<目标分支>
审核人：<审核人列表或无>
源分支将在合并后自动删除。

💡 提示：
- 若之前暂存了变更，可执行 `git stash pop` 恢复。
- MR 合并后，本地可执行 `git checkout main && git pull` 更新主分支。
```

若推送成功但未返回 MR 链接，提示用户前往 GitLab 项目页面查看。

## 使用示例
```bash
# 标准创建 MR
/git-merge

# 指定目标分支
/git-merge --target develop

# 指定审核人
/git-merge --reviewer @alice @bob

# 同时指定目标分支和审核人
/git-merge -t main -r @tech-lead
```

## 注意事项
- GitLab 版本需 ≥ 11.10，Git 版本需 ≥ 2.10（推荐 2.18+）。
- 描述中的特殊字符（如双引号、换行）需由指令自动转义，确保命令行有效。
- 若 `CODEOWNERS` 文件无法读取或无匹配，审核人字段将留空（GitLab 会使用项目默认规则）。
- 本指令会调用 `/git-push` 和 `/git-stash`，请确保对应指令文件存在。

