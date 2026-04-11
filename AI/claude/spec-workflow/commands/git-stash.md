请将当前工作区的未提交变更保存到 Git Stash 中，以便临时切换上下文。

## 执行步骤

### 1. 检查是否有可暂存的变更
- 执行 `git status --porcelain` 检查工作区和暂存区状态。

- 若无任何变更，输出提示：

  ```txt
  ℹ️ 工作区干净，无需暂存。
  ```

流程结束。

### 2. 智能生成 Stash 消息
分析当前未提交的变更内容（包括已暂存和未暂存），提取：
- 改动类型`type`：`feat` / `fix` / `refactor` / `docs` / `style` / `test` / `chore` / `perf` / `ci`
- 识别当前切换的分支名`scope`，比如`xxx/main`，`xxx/xxx/3.2`，则取`/`的最后名称作为分支来源，即：`main`、`3.2`等分支名或版本标识。
- 关键描述`description`：从变更代码中提炼简短摘要（如“修复权限校验”、“添加头像上传组件”）

生成建议消息，格式：

```txt
<type>(<scope>): <description>
```

若无明显特征，则以带时间戳的默认消息代替：`WIP: 临时保存于 YYYY-MM-DD HH:MM`

### 3. 用户确认消息
向用户展示建议消息，并允许修改：

```txt
请输入 Stash 描述（用于在 `git stash list` 中识别）：
建议：<生成的建议消息>

（直接回车使用建议消息，或输入自定义描述）
```

### 4. 执行暂存
- 使用用户确认后的消息执行：`git stash push -m "<消息>"`
- 默认包含未跟踪文件：`git stash push -u -m "<消息>"`（推荐）
- 若用户希望仅暂存已跟踪文件，可加参数 `--skip-untracked`

### 5. 输出结果与后续提示

```txt
✅ 已暂存变更到 stash@{0}
消息：<消息>

后续操作提示：
- 查看 stash 列表：git stash list
- 恢复最近一次 stash：git stash pop
- 恢复但不删除 stash：git stash apply
- 清空所有 stash：git stash clear
```

## 使用示例
```bash
# 基本用法
/git-stash

# 指定自定义消息
/git-stash --message "临时保存：重构订单模块"

# 仅暂存已跟踪文件
/git-stash --skip-untracked
```

## 注意事项
- Stash 是本地操作，不会影响远程仓库。
- 多次 stash 会形成栈结构，可通过 `git stash list` 查看历史。
- 恢复时若产生冲突，需手动解决后执行 `git stash drop` 清理。