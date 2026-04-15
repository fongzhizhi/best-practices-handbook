---
description: 通过 GitLab API 创建 MR。所需参数均由调用方传入或通过 gitlab-config Skill 获取。
---

## 前置条件
- 已配置 `gitlab-config` Skill。
- `.claude/settings.json` 中包含有效的 `gitlab.token` 和 `gitlab.host`。

## 执行逻辑

1. **获取参数**：调用方必须提供以下 JSON 格式的参数（可通过标准输入或变量传入）：
{
    "source_branch": "当前分支名",
    "target_branch": "目标分支名（可选，若未提供则由 Skill 决策）",
    "title": "MR 标题",
    "description": "MR 描述",
    "reviewer_usernames": ["alice", "bob"]
}

2. **补全配置**：
   - 若未提供 `target_branch`，调用 `gitlab-config` Skill 的“确定目标分支”能力获取。
   - 若提供了 `reviewer_usernames` 但非数字 ID，调用 `gitlab-config` Skill 的“用户名转 ID”能力转换。

3. **读取连接信息**：调用 `gitlab-config` Skill 获取 `token` 和 `host`。

4. **构造并执行 API 请求**：
   
   ```bash
   curl -X POST "$GITLAB_HOST/api/v4/projects/$PROJECT_ID/merge_requests" \
     -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "source_branch": "'"$SOURCE_BRANCH"'",
       "target_branch": "'"$TARGET_BRANCH"'",
       "title": "'"$TITLE"'",
       "description": "'"$DESCRIPTION"'",
       "assignee_ids": ['"$ASSIGNEE_IDS"'],
       "remove_source_branch": true
     }'
   ```
   
   其中 `$PROJECT_ID` 可通过解析 `git remote get-url origin` 后调用 API 获取。
   
5. **输出结果**：
   - 成功时输出 MR 的 `web_url`。
   - 失败时输出 `❌ MR_CREATE_FAILED: <API 错误信息>`。

## 注意事项
- 本指令不进行任何交互询问，所有缺失必需参数将导致直接失败。
- 调用方（如 `implementer` Agent）负责在调用前通过 `gitlab-config` Skill 收集好参数。
