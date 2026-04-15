请通过 GitLab API 为当前分支创建合并请求（Merge Request），实现真正的一键创建，无需手动确认。

## 执行步骤

### 1. 环境检查与 Token 获取

#### 1.1 读取 GitLab Token
- **从 Claude Code 配置中读取 `gitlab.token`**：检查当前项目的 `.claude/settings.json` 或用户全局的 `~/.claude/settings.json`，从中提取 `gitlab.token` 字段的值。
- 若未找到该配置，输出以下提示并终止流程：
  ```txt
  ❌ 未找到 GitLab Personal Access Token 配置。
  
  请在 Claude Code 设置文件中添加 Token（任选其一）：
  
  用户全局配置（推荐，所有项目通用）：
  文件位置：C:\Users<用户名>.claude\settings.json
  添加内容：
  {
  "gitlab": {
  "token": "你的个人访问令牌"
  }
  }
  
  当前项目配置（仅当前项目生效）：
  文件位置：<项目根目录>.claude\settings.json
  添加相同内容。
  
  Token 创建指引：登录 GitLab → Settings → Access Tokens，勾选 api 和 write_repository 权限。
  配置完成后，请重新运行 /git-merge。
  ```

**流程终止**。

#### 1.2 确定 GitLab 服务器地址
- 优先从配置中读取 `gitlab.host`（与 `token` 同级）。
- 若未配置，则从 `git remote get-url origin` 中解析（支持 HTTP/SSH 格式）。
- 解析失败则询问用户输入（例如 `https://gitlab.example.com`）。

### 2. 状态检测与前置处理

- 执行 `git status --porcelain` 检查工作区和暂存区状态。
- 执行 `git branch --show-current` 获取当前分支名 `SOURCE_BRANCH`。
- 执行 `git rev-parse --abbrev-ref @{upstream} 2>/dev/null` 检查上游跟踪分支。
- 执行 `git log --oneline origin/$SOURCE_BRANCH..HEAD 2>/dev/null` 检查是否有未推送的本地提交。

**未提交变更处理**：
若存在未提交变更（工作区或暂存区非空），提示用户：
  ```txt
⚠️ 当前有未提交的变更，创建 MR 前需先提交并推送。
请选择处理方式：
A. 自动提交并推送（调用 /git-push）
B. 暂存变更并继续（调用 /git-stash）
C. 取消操作
  ```
- 选 A：调用 `/git-push` 完成提交和推送，继续后续步骤。
- 选 B：调用 `/git-stash` 暂存变更，继续后续步骤。
- 选 C：终止流程。

**未推送提交处理**：
若存在未推送的本地提交（或未设置上游），提示用户：
```txt
ℹ️ 当前分支有未推送的提交，创建 MR 前需先推送到远程。
是否立即推送？(y/n)
```
- 选 y：调用 `/git-push` 完成推送。
- 选 n：终止流程。

### 3. 获取项目信息与默认分支
- 从 `git remote get-url origin` 提取项目路径（例如 `namespace/project`）。
- 对项目路径进行 URL 编码（将 `/` 替换为 `%2F`）。
- 调用 GitLab API 获取项目 ID 和默认分支：
  ```bash
  PROJECT_PATH_ENCODED=$(git remote get-url origin | sed -e 's/.*[:/]\(.*\)\.git/\1/' -e 's/\//%2F/g')
  curl -s -H "PRIVATE-TOKEN: $GITLAB_TOKEN" "$GITLAB_HOST/api/v4/projects/$PROJECT_PATH_ENCODED"
- 从返回的 JSON 中提取 `id`（项目 ID）和 `default_branch`。
- 若 API 返回错误，提示检查 Token 权限和项目路径，终止流程。

### 4. 确定目标分支
按以下优先级：
1. 用户通过 `--target` 或 `-t` 参数指定的分支。
2. 项目 `.claude/config.json` 中的 `mr.defaultTargetBranch`。
3. 上一步获取的 `default_branch`。
4. 若以上均无法确定，询问用户输入目标分支名。

### 5. 确定审核人（Assignee）
GitLab API 的 `assignee_ids` 字段需要用户数字 ID。按以下步骤处理：
1. 用户通过 `--reviewer` 或 `-r` 参数指定的分支。
2. 检查当前项目的 `.claude/settings.json` 或用户全局的 `~/.claude/settings.json`，从中提取 `mr.defaultReviewers` 字段的值。
3. 若未获取到任何用户名，则跳过审核人设置。
4. 若有用户名，对每个用户名调用 API 获取 ID：
   ```bash
   curl -s -H "PRIVATE-TOKEN: $GITLAB_TOKEN" "$GITLAB_HOST/api/v4/users?username=$USERNAME"
   
   从返回 JSON 中提取第一个对象的 `id`。若失败，该用户将被忽略。

### 6. 生成 MR 标题和描述
**标题与描述的获取优先级**：

1. 根据历史对话和变更信息，总结merge request提交信息。
1. 注意使用中文描述

**消息格式**：

- **标题**：`<type>(<scope>): <简短描述>`。
- **描述**：包含以下部分（若有则填写）：

  ```markdown
  ## 变更说明
  <变更信息总结>
  
  ## 关联信息
  - ONES: <ONES单ID> <ONES单标题>
  - Spec: <Spec文档路径>
  - Link: <ONES单URL>
  ```
  ONES 和 Spec 信息从当前对话上下文或最近 commit 的尾部标记中提取。描述中的双引号和换行符需正确转义。

### 7. 调用 API 创建 MR
构造 JSON 请求体并发送 POST 请求：

```bash
curl -X POST "$GITLAB_HOST/api/v4/projects/$PROJECT_ID/merge_requests" \
  -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"source_branch\": \"$SOURCE_BRANCH\",
    \"target_branch\": \"$TARGET_BRANCH\",
    \"title\": \"$TITLE\",
    \"description\": \"$DESCRIPTION\",
    \"assignee_ids\": [$ASSIGNEE_IDS],
    \"remove_source_branch\": true
  }"
```

- `$ASSIGNEE_IDS` 为逗号分隔的用户数字 ID（例如 `123,456`）。
- 若 `ASSIGNEE_IDS` 为空，则省略 `assignee_ids` 字段。

### 8. 输出结果
- 解析 API 返回的 JSON，提取 `web_url` 字段。
- 输出成功信息：
  ```txt
  ✅ 合并请求已创建
  链接：<MR 的 Web URL>
  目标分支：<目标分支>
  审核人：<用户名列表或无>
  源分支将在合并后自动删除。
  ```
- 若 API 返回错误（如 `{"message":"..."}`），显示错误详情，并提示用户可手动创建。

### 9. 后续操作提示
```txt
💡 提示：
- 若之前暂存了变更，可执行 `git stash pop` 恢复。
- MR 合并后，本地可执行 `git checkout <目标分支> && git pull` 更新。
```

## 使用示例
```bash
/git-merge                       # 自动创建 MR
/git-merge --target develop      # 指定目标分支
/git-merge --reviewer @alice @bob  # 指定审核人
```

## 注意事项
- Token 需具备 `api` 权限，请勿将 Token 提交至代码仓库。
- 若内网 GitLab 使用自签名证书，请在 curl 命令中添加 `-k` 参数（临时）或将证书加入信任链。
- 本指令依赖 `/git-push` 和 `/git-stash`，请确保对应指令文件存在。