---
description: 执行 git stash。若未提供 message，默认使用 "WIP"。
---

运行：`git stash push -m "${ARGUMENTS:-WIP}"`

若命令失败，输出 `❌ GIT_STASH_FAILED: <原始错误>`。