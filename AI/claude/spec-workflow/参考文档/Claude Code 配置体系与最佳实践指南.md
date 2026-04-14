---
createDate: 2026-04-14
---

# Claude Code 配置体系与最佳实践指南

---

## 一、核心概念对比：一张表理清

| 概念                    | 定义                                                 | 存放位置                           | 触发方式                                                     | 适用场景                                                     |
| :---------------------- | :--------------------------------------------------- | :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **CLAUDE.md**           | 项目级全局指令文件，每次对话启动时自动加载           | 项目根目录或 `~/.claude/CLAUDE.md` | **自动加载**（每轮对话）                                     | 项目架构、技术栈、常用命令、编码规范等“永不改变”的基础信息   |
| **Rules**               | 可拆分的规则文件，支持按路径过滤加载                 | `.claude/rules/`                   | **自动加载**（可配置 `paths` 按需加载）                      | 将大而全的 CLAUDE.md 拆分为编码规范、测试规范、Git 规范等独立文件 |
| **Skills**              | 可复用的知识包，支持渐进式披露                       | `.claude/skills/<name>/SKILL.md`   | **模型调用**（Claude 根据描述自动判断是否激活）或 **用户显式调用** | 需要特定领域知识的任务，如你的 `spec-miner-openspec` 技能    |
| **Commands**            | 自定义斜杠命令，用户显式输入触发                     | `.claude/commands/`                | **用户手动输入** `/command-name`                             | 执行重复性任务，如你的 `/git-checkout`、`/git-commit` 等 Git 自动化指令 |
| **Agents（Subagents）** | 具有独立配置（工具白名单、模型、权限模式）的子智能体 | `.claude/agents/<name>.md`         | **Task 工具调用**或 **用户显式调用**                         | 将复杂任务分解为专业化子任务，在隔离上下文中并行执行         |
| **Hooks**               | 生命周期事件触发执行的自动化脚本或提示               | `settings.json` 中的 `hooks` 字段  | **自动触发**（14个生命周期事件）                             | 强制执行的确定性规则，如每次编辑后自动运行 ESLint、禁止写入敏感目录 |


## 二、各概念深入解析与最佳实践

### 2.1 CLAUDE.md —— 你的“Agent 入职手册”

**它做什么**：Claude Code 在每次对话开始前，第一件事就是读取 `CLAUDE.md` 文件，将其内容注入系统提示。这意味着你在这里写的内容，Claude 在任何代码操作之前就已经知道了。

**加载层级**：Claude Code 按以下优先级（从高到低）发现并加载 CLAUDE.md：
1. 企业级托管策略（`/Library/Application Support/ClaudeCode/CLAUDE.md`）
2. 用户全局（`~/.claude/CLAUDE.md`）——你的个人偏好
3. 项目根目录（`./CLAUDE.md` 或 `./.claude/CLAUDE.md`）——团队共享，提交到 Git
4. 本地项目（`./CLAUDE.local.md`）——个人项目级设置，不提交
5. 父目录（向上遍历加载）——Monorepo 场景下自动加载上层 CLAUDE.md
6. 子目录（按需加载）——当 Claude 读取该目录下文件时才加载

**最佳实践**：
- 在 `CLAUDE.md` 中只放“删掉就会导致 Claude 犯错”的信息：构建命令、测试命令、架构边界、非标准编码规范。不要放 Claude 能从代码中自己推断出的内容。
- 使用 `@.claude/rules/workflow-router.md` 语法引用其他规则文件，实现模块化管理。
- 200 行是推荐的最大行数，超出后 Claude 的遵守率会下降。

### 2.2 Rules —— 拆分大文件的模块化方案

**它做什么**：当 `CLAUDE.md` 越来越长时，可以用 `.claude/rules/` 目录将规则按领域拆分。每个 `.md` 文件都可以通过 YAML frontmatter 中的 `paths` 字段指定加载条件。

**示例**：
```markdown
---
paths: "src/api/**"
---
# API 设计规范
所有 API 路由必须返回 Result<T, E> 类型，不允许直接 throw。
```

**最佳实践**：
- 你的 `workflow-router.md` 和 `constitution.md` 放在 `.claude/rules/` 是正确做法。
- 建议拆分为：`coding-style.md`、`testing.md`、`git-workflow.md`、`security.md` 等。
- 设置 `paths` 字段可以实现“修改 API 代码时自动加载 API 规范，修改前端代码时不加载”，节省 token。

### 2.3 Skills —— 模型自动调用的知识包

**它做什么**：Skills 是一组结构化的指令，用于教导 Claude 如何处理特定任务。与 Commands 最大的不同是：**Skills 是模型调用的**——Claude 根据对话内容自动判断是否需要启用某个 Skill。

**两种使用模式**：

| 模式                  | 加载方式                                  | 调用方式                                 | 适用场景                    |
| :-------------------- | :---------------------------------------- | :--------------------------------------- | :-------------------------- |
| **Standalone Skills** | 按需加载                                  | 用户输入 `/skill-name` 或 Skill 工具调用 | 用户主动触发的可复用工作流  |
| **Agent Skills**      | Agent 启动时预加载（通过 `skills:` 字段） | 自动注入 Agent 上下文                    | 作为特定 Agent 的“背景知识” |

**关键 YAML 字段**：
- `description`：这是 Skill 的“灵魂”，Claude 靠这段描述决定何时激活该 Skill。
- `user-invocable: false`：设置后该 Skill 不显示在 `/` 菜单中，只能作为 Agent 的背景知识。
- `allowed-tools`：限制该 Skill 激活时允许自动执行的工具，减少权限弹窗。
- `context: fork`：在隔离的子 Agent 中运行该 Skill，避免污染主对话上下文。

**与你的工作流结合**：你的 `spec-miner-openspec` 是一个典型的 Standalone Skill。用户通过自然语言触发，执行分析后自动将产物迁移到 `openspec/specs/`。这正是 Skill 的最佳使用场景——需要领域知识、可复用、可由模型根据描述自动激活。

### 2.4 Commands —— 你的自动化指令库

**它做什么**：Commands 是 Markdown 文件，定义了可通过 `/command-name` 调用的可复用提示词。每个 `.claude/commands/` 目录下的 `.md` 文件自动成为一个斜杠命令。

**最佳实践**：
- 你的 `/git-*` 系列指令全部是 Commands。这是正确的选择——Git 操作是用户主动触发的行为，用 Commands 符合心智模型。
- 支持 `$ARGUMENTS` 变量注入动态参数，例如 `/git-merge --target develop` 中的 `--target` 可以通过 `$ARGUMENTS` 捕获。
- Commands 可以嵌套在子文件夹中，通过 `/folder/command` 访问。建议你的 Git 指令放在 `.claude/commands/git/` 下，更整洁。

### 2.5 Agents（Subagents）—— 你还没用上的“角色工程化”能力

**它做什么**：Subagents 是具有独立配置（工具白名单、模型选择、权限模式、预加载 Skills、MCP 服务器）的自定义 AI Agent。它们通过 `Task` 工具被主 Agent 调用，在隔离上下文中并行执行任务。

**为什么你现在还没用但值得了解**：
- Subagents 的定位是“把固定角色固化成可复用配置，让主对话负责编排，子代理负责专业产出”。
- 每个 Subagent 都有独立的上下文窗口，不会污染主对话，适合处理大型搜索、深度分析等任务。

**与你的工作流可能结合的场景**：
- 可以创建一个 `code-reviewer` Subagent，专门做代码审查，输出分级意见。
- 可以创建一个 `spec-validator` Subagent，专门检查 Spec 文档与代码的一致性（替代你之前想做的 Drift Check）。
- 每个 Subagent 可以预加载不同的 Skills，例如 `spec-miner-openspec` 技能可以预加载给一个 `legacy-code-analyzer` Subagent。

**配置示例**：
```markdown
---
name: code-reviewer
description: 专门审查代码变更，输出分级审查意见
model: haiku
tools: ["Read", "Grep", "Glob"]
skills:
  - constitution
permissionMode: plan
---
你是一个代码审查专家。审查变更时，请对照项目宪法（constitution.md）进行检查，输出：
- 必须修改的问题
- 建议修改的问题
- 可选优化项
```

### 2.6 Hooks —— 确定性的自动化守卫

**它做什么**：Hooks 是在 Claude Code 生命周期的特定节点（14 个事件）自动触发的 Shell 命令或 LLM 提示。

**与 CLAUDE.md 的关键区别**：CLAUDE.md 是“建议性的”——Claude 读了但可能在长会话中忘记。Hooks 是“确定性的”——每次都会执行，零例外。

**适用场景**：
- 每次编辑文件后自动运行 ESLint（`PreToolUse` 或 `PostToolUse` 事件）
- 提交前自动运行 `/pre-commit` 检查（`PreToolUse` 事件，监听 `Bash(git commit)`）
- 禁止向 `migrations/` 目录写入（`PreToolUse` 事件，监听 `Edit` 工具）

**为什么你的工作流可以考虑 Hooks**：
- 你设计的 `/pre-commit` 指令需要用户手动运行。如果用 Hooks 在每次 `git commit` 前自动执行，可以彻底避免漏检。
- 你可以在 `settings.json` 中配置 Hooks，实现强制性的流程管控，例如“主分支禁止直接推送”这类规则用 Hook 比用 Command 更可靠。


## 三、你的工作流中的层级架构与最佳实践

基于以上概念，你的工作流已经形成了一个清晰的三层架构：

```
┌─────────────────────────────────────────────────────────────────┐
│  决策层（CLAUDE.md + Rules）                                     │
│  - CLAUDE.md：项目架构、技术栈、常用命令                          │
│  - constitution.md：编码规范、命名、错误处理                      │
│  - workflow-router.md：场景识别与工作流路由                       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  执行层（Commands + Skills）                                     │
│  - /git-* 系列：Git 操作自动化（Commands）                       │
│  - /pre-commit：质量检查（Command）                              │
│  - spec-miner-openspec：逆向规范提取（Skill）                    │
│  - /openspec:*：规范驱动开发（OpenSpec 提供的 Commands）          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  保障层（Hooks + Subagents）—— 你尚未启用但可考虑的扩展           │
│  - Hooks：强制执行质量门禁、阻断危险操作                          │
│  - Subagents：将代码审查、Spec 验证等任务专业化解耦               │
└─────────────────────────────────────────────────────────────────┘
```

### 最佳实践清单

| 层级          | 已做到的                                         | 可优化的                                                     |
| :------------ | :----------------------------------------------- | :----------------------------------------------------------- |
| **CLAUDE.md** | ✅ 引用 `constitution.md` 和 `workflow-router.md` | 建议限制在 200 行以内，定期审视删除冗余内容                  |
| **Rules**     | ✅ 拆分了编码规范和工作流引导                     | 可进一步按领域拆分（testing.md、git.md），并为高频模块配置 `paths` 按需加载 |
| **Commands**  | ✅ `/git-*` 系列智能独立可用                      | 可放入 `.claude/commands/git/` 子目录，通过 `/git:checkout` 调用，更整洁 |
| **Skills**    | ✅ `spec-miner-openspec` 正确封装                 | 可为 Skill 添加 `allowed-tools` 字段减少权限弹窗             |
| **Subagents** | ❌ 尚未使用                                       | 可创建 `code-reviewer` 和 `spec-validator` 两个 Subagent，解耦审查职责 |
| **Hooks**     | ❌ 尚未使用                                       | 可配置 `PostToolUse` Hook 在每次文件编辑后自动运行 Prettier；配置 `PreToolUse` Hook 阻断主分支直接推送 |
| **Settings**  | ✅ 已配置 `curl:*` 和 `git:*` 权限                | 建议将敏感配置（Token 等）放在 `~/.claude/settings.json` 而非项目级 |


## 四、推荐目录结构

```
项目根目录/
├── CLAUDE.md                          # 项目架构、技术栈、命令索引
├── .claude/
│   ├── settings.json                  # 权限配置（git:*, curl:*）
│   ├── commands/
│   │   ├── git/
│   │   │   ├── checkout.md
│   │   │   ├── branch.md
│   │   │   ├── commit.md
│   │   │   ├── push.md
│   │   │   ├── merge.md
│   │   │   └── stash.md
│   │   ├── prettier-format.md
│   │   └── pre-commit.md
│   ├── skills/
│   │   └── spec-miner-openspec/
│   │       └── SKILL.md
│   ├── rules/
│   │   ├── constitution.md            # 编码规范
│   │   ├── workflow-router.md         # 场景识别与引导
│   │   ├── testing.md                 # 测试规范（建议补充）
│   │   └── git-workflow.md            # 分支命名、commit 格式（建议补充）
│   └── agents/                        # 建议补充
│       ├── code-reviewer.md           # 代码审查子 Agent
│       └── spec-validator.md          # Spec 一致性验证子 Agent
└── openspec/                          # OpenSpec 规范库
    ├── specs/
    └── changes/
```


## 五、下一步建议

1. **立即可做**：将 Git 指令移入 `.claude/commands/git/` 子目录，保持整洁。
2. **短期可做**：为 `spec-miner-openspec` Skill 添加 `allowed-tools` 字段，减少权限弹窗。
3. **中期可探索**：创建 `code-reviewer` Subagent，将代码审查从主对话中解耦，避免审查意见污染编码上下文。
4. **长期可考虑**：用 Hooks 实现 `git push` 前自动运行 `/pre-commit`，以及阻断主分支直接推送，让流程保障从“建议”变为“强制”。

这套架构的核心哲学是：**CLAUDE.md 定义“我们是谁”，Rules 定义“我们怎么做”，Commands 提供“我们能做什么”，Skills 注入“我们知道什么”，Hooks 确保“我们一定做”，Subagents 实现“我们分工做”。** 你的工作流已经覆盖了前三层，后续可以根据团队成熟度逐步引入后两层。