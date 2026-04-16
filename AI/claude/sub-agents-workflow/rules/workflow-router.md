# 工作流智能引导规则

本文件为 **Orchestrator Agent** 提供场景识别与调度决策依据。  
当用户输入意图模糊时，Orchestrator 应参考此规则主动推荐工作流。

## 一、场景识别映射

| 触发关键词 / 意图                     | 任务类型   | 推荐工作流                                                   |
| :------------------------------------ | :--------- | :----------------------------------------------------------- |
| 新功能、新增、开发、实现、feat        | 新功能开发 | OpenSpec 提案 → 实施 → 提交 MR                               |
| Bug、修复、问题、异常、报错、defect   | Bug 修复   | 分析定位 → 创建修复提案 → 实施 → 提交 MR                     |
| 重构、整理、优化结构、拆分、rename    | 代码重构   | 提取现状规范 → 创建重构提案 → 逐步实施                       |
| 分析、梳理、理解、查看逻辑、定位原因  | 代码分析   | 调用 Analyzer 输出分析报告                                   |
| 写规范、生成提案、创建 spec、proposal | 规范撰写   | 调用 Spec-Writer 生成 OpenSpec 文件                          |
| 提交、推送、合并、创建 MR、commit     | 代码交付   | 调用 Implementer 执行 Git 流程                               |
| cherry-pick、拣选、把提交合入         | 提交拣选   | 调用 Implementer 执行 cherry-pick 流程                       |
| 格式化、检查、lint、prettier          | 质量保障   | 直接调用 `/quality:prettier-format` 或 `/quality:pre-commit` |

## 二、完整工作流步骤指引

### 2.1 新功能开发（含 OpenSpec）
1. **需求确认**：Orchestrator 明确功能范围和 ONES 单号（若关联）。
2. **分支准备**：调用 `/git:branch`（分支名由 Orchestrator 按规范生成，可借助 `ones-parser` Skill）。
3. **规范编写**：调度 `spec-writer` Agent，生成 `proposal.md` 和 `tasks.md`。
4. **编码实施**：调度 `implementer` Agent，按 `tasks.md` 逐项实现。
5. **质量检查**：`implementer` 内部调用 `/quality:pre-commit`。
6. **提交推送**：`implementer` 依次调用 `/git:commit`、`/git:push`。
7. **创建 MR**：`implementer` 调用 `/git:merge`（参数由 `gitlab-config` Skill 提供）。
8. **规范归档**：MR 合并后，提示用户可执行 `/opsx:archive`。

### 2.2 Bug 修复（含逆向分析）
1. **问题定位**：调度 `analyzer` Agent，调用 `spec-miner` Skill 分析相关模块。
2. **创建提案**：将分析报告传递给 `spec-writer` Agent，生成修复提案。
3. **后续步骤**：与新功能开发的第 3–7 步相同。

### 2.3 代码重构
1. **现状提取**：调度 `analyzer` 调用 `spec-miner` 生成当前行为规范。
2. **设计提案**：调度 `spec-writer` 根据重构目标编写变更提案。
3. **实施与提交**：同新功能开发步骤。

### 2.4 纯代码分析
1. 调度 `analyzer` Agent，直接返回结构化分析报告。
2. 不产生任何代码变更或规范文件。

### 2.5 提交拣选（Cherry-pick）
1. **参数确认**：Orchestrator 从用户输入中提取 `COMMIT_SHA` 和 `TARGET_BRANCH`，若缺失则询问。
2. **调度实施**：调度 `implementer` Agent，调用 `/git:cherry-pick` 指令，传入必要参数。
3. **冲突处理**：若 cherry-pick 发生冲突，`implementer` 暂停并报告 Orchestrator，Orchestrator 提示用户手动解决冲突。
4. **创建 MR**：`implementer` 调用 `/git:merge` 创建 MR，若 API 失败则输出手动创建链接。
5. **结果汇总**：Orchestrator 向用户报告操作结果或手动链接。

## 三、与 Orchestrator 的协作约定

- **Orchestrator 职责**：解析用户意图、补全上下文信息（如 ONES 单号、commit SHA）、按本规则选择工作流、调度子 Agent、向用户汇总结果。
- **子 Agent 职责**：接收明确的任务描述和上下文，返回结构化结果或执行状态。
- **规则更新**：若新增任务类型，需同步更新本文件中的映射表和步骤指引。

## 四、异常处理提示

- **意图不明**：Orchestrator 应列举可能的任务类型供用户选择。
- **前置条件缺失**（如未配置 GitLab Token）：Orchestrator 应提示用户完成配置后再继续。
- **子 Agent 失败**：Orchestrator 应提取错误信息，向用户说明原因并询问后续动作（重试/跳过/人工介入）。
- **Cherry-pick 冲突**：提示用户手动解决冲突，并提供当前分支信息以便继续操作。
