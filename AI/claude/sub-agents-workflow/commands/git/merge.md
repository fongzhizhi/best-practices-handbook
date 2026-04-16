---
description: 通过 GitLab API 创建 MR。若 API 失败，输出手动创建链接供用户自行操作。
---

## 执行逻辑

### 1. 前置条件
- `.claude/settings.json` 中需配置 `gitlab.token` 和 `gitlab.host`。
- 调用方需传入 JSON 格式参数，包含：
  - `source_branch`（源分支）
  - `target_branch`（目标分支）
  - `title`（MR 标题）
  - `description`（MR 描述，可选）
  - `reviewer_ids`（审核人数字 ID 数组，可选）

### 2. 参数处理
从 `gitlab-config` Skill 或调用方传入的 JSON 中提取以下变量：
- `GITLAB_HOST`
- `GITLAB_TOKEN`
- `PROJECT_ID`（需从当前仓库的 remote URL 推断，或由调用方传入）
- `SOURCE_BRANCH`
- `TARGET_BRANCH`
- `TITLE`
- `DESCRIPTION`
- `REVIEWER_IDS`

若 `PROJECT_ID` 未知，可通过 `git remote get-url origin` 解析出项目路径（如 `group/project`），URL 编码后用于 API。

### 3. 创建 MR（API 调用）
构造 JSON 体，使用 `jq` 或直接拼接，确保中文被正确转义（避免手动拼接引号）。推荐使用 `jq` 生成安全 JSON：

```bash
BODY=$(jq -n \
  --arg source "$SOURCE_BRANCH" \
  --arg target "$TARGET_BRANCH" \
  --arg title "$TITLE" \
  --arg desc "$DESCRIPTION" \
  --argjson reviewers "$REVIEWER_IDS" \
  '{
    source_branch: $source,
    target_branch: $target,
    title: $title,
    description: $desc,
    reviewer_ids: $reviewers
  }')
```

执行 curl（设置超时 30s，连接超时 10s）：

```bash
RESPONSE=$(curl -s -w "\n%{http_code}" --max-time 30 --connect-timeout 10 \
  -X POST "$GITLAB_HOST/api/v4/projects/$PROJECT_ID/merge_requests" \
  -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  --data-binary "$BODY")
```

从响应中分离 HTTP 状态码和响应体：
```bash
HTTP_CODE=$(echo "$RESPONSE" | tail -n1)
BODY=$(echo "$RESPONSE" | sed '$d')
```

### 4. 结果判断与降级

#### 4.1 成功（HTTP 200/201）
从响应体提取 `web_url`，输出：
```
✅ MR 创建成功：<web_url>
```

#### 4.2 失败（非 2xx 或 curl 错误）
1. **输出错误信息**：`❌ MR_CREATE_FAILED: HTTP $HTTP_CODE - $(echo "$BODY" | jq -r '.message // "Unknown error"' 2>/dev/null)`
2. **生成手动创建链接**：
   ```bash
   MANUAL_URL="$GITLAB_HOST/$(git remote get-url origin | sed 's/.*[:/]\(.*\)\.git/\1/')/-/merge_requests/new?merge_request%5Bsource_branch%5D=$SOURCE_BRANCH&merge_request%5Btarget_branch%5D=$TARGET_BRANCH"
   ```
   对 `TITLE` 和 `DESCRIPTION` 进行 URL 编码后可附加到链接中（可选）。

3. **输出降级提示**：
   ```
   ⚠️ 自动创建 MR 失败，请手动创建：
   🔗 <MANUAL_URL>
   ```

### 5. 清理与退出
- 成功时返回 0，失败时返回 1（便于上层 Agent 判断）。

## 依赖工具
- `curl`
- `jq`（用于 JSON 构造和解析，若未安装可降级为手动拼接，但需处理中文转义）
- `git`（用于获取 remote URL）

## 注意事项
- 使用 `--data-binary` 而非 `--data` 可避免 curl 对中文进行不必要的转码。
- 若 `jq` 不可用，可简化 JSON 构造（但需确保 `TITLE` 和 `DESCRIPTION` 中的双引号和换行符被正确转义）。若无 `jq`，建议在指令中检查并提示安装。
