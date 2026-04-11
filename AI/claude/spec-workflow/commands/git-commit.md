请根据当前分支的变更内容，生成一条符合 Conventional Commits 规范的 commit message，并自动附加对话上下文中的 ONES 单信息和 Spec 文档信息，然后执行提交。

## 执行步骤

### 1. 获取代码变更内容
- 执行 `git diff --cached` 获取暂存区变更；若暂存区为空，则执行 `git diff` 获取工作区变更。
- 分析变更内容，识别：
  - 改动类型：feat / fix / refactor / docs / style / test / chore / perf / ci
  - 影响范围（scope），如 auth、editor、payment
  - 关键变更点摘要（用于 commit body）

### 2. 从对话上下文中提取 ONES 信息
扫描当前对话历史，识别 ONES 单的特征：
- **链接格式**：包含 `ones.` 的 URL（如 `http://ones.lceda/xxx`）
- **ID 格式**：`[A-Z]+-\d+`，如 `PRO-151081`
- **标题格式**：紧跟在 ID 后的中文描述，如 `【编辑器】编辑器打开PCB报错，提示无权限`

若存在多个 ONES 单，选择最近一次提及或与变更最相关的。若未发现则留空。

### 3. 从对话上下文中提取 Spec 信息
扫描对话历史，识别 Spec 文档引用：
- 路径格式：`openspec/specs/` 下的 `.md` 文件，或 `openspec/changes/` 下的目录名
- 优先提取最近生成或更新的 Spec 路径。若未发现则留空。

### 4. 生成 commit message
采用以下格式：

```txt
<type>(<scope>): <简短描述>

<详细变更点列表，每行以 - 开头>

ONES: <ONES单ID> <ONES单标题>
Spec: <Spec文档相对路径>
Link: <ONES单URL>
```

示例：

```txt
fix(editor): 修复PCB打开时的权限校验错误

- 将权限检查前置到文件读取之前
- 增加无权限时的友好提示
- 补充权限拒绝的错误处理

ONES: PRO-151081 【编辑器】编辑器打开PCB报错，提示无权限
Spec: openspec/specs/editor/permission/spec.md
Link: http://ones.lceda/xxxx
```

若无 ONES/Spec，则省略相应行。

### 5. 用户确认并提交
- 向用户展示完整的 commit message 预览。
- 询问用户：“是否使用此消息提交？(y/n)”
- 若用户回复 `y`，则执行 `git add .`（若暂存区为空）并将该消息写入 commit：`git commit -m "生成的消息"`。
- 若用户回复 `n`，则允许用户手动修改后再执行或取消提交。

## 注意事项
- 若同时存在多个 ONES 单，应向用户确认关联哪一个，或提示用户手动指定。
- 若无法从上下文自动提取，用户可在命令中手动传入，如：
  `/git-commit --ones PRO-151081 --spec openspec/specs/editor/spec.md`
