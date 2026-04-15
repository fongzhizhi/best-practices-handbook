---
name: gitlab-config
description: 读取 GitLab 相关配置（Token、Host、默认目标分支、审核人映射表），按优先级返回决策结果，并提供用户名到 ID 的转换能力。供 implementer Agent 或 /git-merge 指令调用。
---

# GitLab 配置解析器

本 Skill 封装了所有与 GitLab 配置相关的读取与决策逻辑，确保配置获取的一致性。

## 能力列表

### 1. 获取 GitLab 连接配置
从 `.claude/settings.json` 中读取 `gitlab.token` 和 `gitlab.host`。

- **输入**：无
- **输出**：`{ "token": "...", "host": "..." }`
- **异常**：若未配置，返回错误提示。

### 2. 确定目标分支
按以下优先级返回 MR 的目标分支：
1. 用户通过参数 `--target` 指定的值（由调用方传入）。
2. 项目配置文件 `.claude/config.json` 中的 `mr.defaultTargetBranch`。
3. 仓库默认分支（通过 API 或本地 `git symbolic-ref refs/remotes/origin/HEAD` 获取）。
4. 若以上均无法确定，返回空并提示询问用户。

- **输入**：`user_target` (可选字符串)
- **输出**：目标分支名字符串。

### 3. 确定审核人 ID 列表
按以下优先级返回审核人的数字 ID 数组：
1. 用户通过参数 `--reviewer` 或 `--reviewer-id` 指定的值（由调用方传入）。
2. 项目配置文件 `.claude/config.json` 中的 `mr.defaultReviewers` 数组（支持用户名或数字 ID）。
3. 项目根目录或 `.gitlab/CODEOWNERS` 文件中匹配变更文件的审核人（需解析为 ID）。
4. `.claude/settings.json` 中 `gitlab.userIds` 映射表（用户名到 ID 的映射）。
5. 若以上均未获取到，返回空数组。

- **输入**：`user_reviewers` (可选，用户名或 ID 数组)，`changed_files` (可选，用于 CODEOWNERS 匹配)
- **输出**：审核人数字 ID 数组。

### 4. 用户名转换为数字 ID
对给定用户名列表，首先从本地映射表 `gitlab.userIds` 查找，未命中则调用 GitLab API `GET /api/v4/users?username=<用户名>` 查询。

- **输入**：用户名数组
- **输出**：数字 ID 数组（无法转换的用户将被忽略并记录警告）

## 配置示例（`.claude/settings.json`）

```json
{
  "gitlab": {
    "token": "glpat-xxxxxxxxxxxxxxxxxxxx",
    "host": "https://gitlab.internal.com",
    "userIds": {
      "alice": 123,
      "bob": 456,
      "tech-lead": 789
    }
  },
  "mr": {
    "defaultTargetBranch": "develop",
    "defaultReviewers": ["alice", "bob"]
  }
}
```

## 使用说明

Agent 或 Command 在需要 GitLab 配置时，调用本 Skill 的对应函数即可，无需关心具体获取细节。