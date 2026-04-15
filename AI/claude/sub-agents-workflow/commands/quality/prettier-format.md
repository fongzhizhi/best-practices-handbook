---
description: 对指定文件执行 Prettier 格式化。文件列表由调用方传入。
---

运行：`npx prettier --write $ARGUMENTS`

若 `$ARGUMENTS` 为空，输出 `❌ NO_FILES_PROVIDED`。
若 Prettier 未安装，输出 `❌ PRETTIER_NOT_FOUND`。
