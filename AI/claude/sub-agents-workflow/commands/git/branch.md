---
description: 创建并切换到新分支。分支名必须由调用方传入，不生成、不迁移 stash。
---

运行：`git checkout -b $ARGUMENTS`

若 `$ARGUMENTS` 为空，询问调用方提供分支名后重试。
若命令失败，输出 `❌ GIT_BRANCH_FAILED: <原始错误>`。
