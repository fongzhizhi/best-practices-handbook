---
description: 执行提交前质量检查。检查项从配置文件读取，仅输出检查结果，不做修复。
---

## 执行逻辑
1. 读取 `.claude/settings.json` 中的 `quality.preCommit` 配置。
2. 若未配置，使用默认检查项：ESLint、TypeScript 类型检查。
3. 依次运行启用的检查命令。
4. 输出 JSON 格式结果：`{"passed": true/false, "errors": [...]}`。
5. 若有失败项，返回非零退出码。
