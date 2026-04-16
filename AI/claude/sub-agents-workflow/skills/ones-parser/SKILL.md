---
name: ones-parser
description: 解析 ONES 单号、标题和链接，生成分支名、commit 尾部标记等标准化输出。当需要从对话或提交中提取 ONES 信息时使用。
---

# ONES 单解析器

本 Skill 封装了 ONES 需求/缺陷单的识别、提取与格式化能力，为分支命名、commit message 生成等环节提供一致的输出。

## 能力列表

### 1. 从对话上下文提取 ONES 信息
扫描当前对话历史，识别以下特征：
- **ID 格式**：`[A-Z]+-\d+`（如 `PRO-151081`）
- **标题格式**：紧跟在 ID 后的中文描述（如 `【编辑器】编辑器打开PCB报错，提示无权限`）
- **链接格式**：包含 `ones.lceda` 或 `ones.xxx` 的 URL

若存在多个 ONES 单，默认选择**最近一次提及**或与当前变更最相关的那个。若未发现任何 ONES 信息，返回空。

### 2. 生成分支名
根据提取的 ONES 信息和调用方提供的上下文，按团队规范生成分支名。

- **输入**：
  - `ones_id`：ONES 单号（必需）
  - `base_branch`：基准分支名（如 `main`、`develop`、`3.2`），默认从当前分支名提取最后一段
  - `change_type`：变更类型（`feat` / `fix` / `refactor` 等），若未提供则根据 ONES 标题推断
- **输出**：`<基准分支>/<变更类型>/<ONES_ID>`
- **示例**：
  - 输入：`ones_id=PRO-151081`，`base_branch=main`，`change_type=fix`
  - 输出：`main/fix/PRO-151081`

### 3. 生成 commit 尾部标记
根据 ONES 信息和可选的 Spec 路径，生成符合团队规范的 commit message 尾部内容。

- **输入**：
  - `ones_id`、`ones_title`、`ones_url`
  - `spec_path`：OpenSpec 规范文档路径（可选）
- **输出格式**：
```
  ONES: <ONES_ID> <ONES_TITLE>
  Spec: <SPEC_PATH>
  Link: <ONES_URL>
```
- **示例**：
  ```
  ONES: PRO-151081 【编辑器】编辑器打开PCB报错，提示无权限
  Spec: openspec/specs/editor/permission/spec.md
  Link: http://ones.lceda/xxxx
  ```
  若无 Spec 或 Link，则省略对应行。

### 4. 补全 ONES 详情（可选）
若仅提供 ONES ID，可提示 Orchestrator 或用户补充标题和链接。本 Skill 本身不主动调用外部 API，但可通过 `Task` 工具请求 Orchestrator 协助获取。

## 使用示例

### 场景一：创建分支前生成分支名
  ```
调用方：Orchestrator
需求：为 ONES 单 PRO-151081 生成功能分支名，基准分支为 main，类型为 feat
Skill 返回：main/feat/PRO-151081
  ```

### 场景二：生成 commit message 尾部
```
调用方：implementer
需求：生成当前 commit 的尾部标记，ONES 单为 PRO-151081，Spec 路径为 openspec/specs/editor/spec.md
Skill 返回：
ONES: PRO-151081 【编辑器】编辑器打开PCB报错
Spec: openspec/specs/editor/spec.md
Link: http://ones.lceda/xxxx
```

## 约束

- 本 Skill 仅提供解析与格式化能力，不修改任何文件。
- 若 ONES 信息不完整，应返回明确提示（如“缺少 ONES 标题”），由调用方决定后续动作。
- 所有输出均遵循团队现有规范，无额外决策逻辑。
