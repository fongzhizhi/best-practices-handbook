---
description: 执行 git commit。提交信息必须由调用方提供且已正确转义。
---

运行：`git commit -m "$ARGUMENTS"`

若 `$ARGUMENTS` 为空或命令失败，输出 `❌ GIT_COMMIT_FAILED: <原始错误>`。
