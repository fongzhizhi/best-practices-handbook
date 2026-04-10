---
createDate: 2026-04-10
---

## OpenSpec 与 Spec Miner 完全上手指南

### 一、两套工具分别是什么

#### OpenSpec：规范驱动开发框架（先写规范，再写代码）

OpenSpec 由 Fission AI 团队开发，是一套面向 AI 编码助手的**规范驱动开发工作流**。它的核心理念是：在写代码之前，人类与 AI 必须先就规范达成共识（align before code）。

需要**通过 npm 全局安装**（Node.js ≥20.19.0）：

```bash
npm install -g @fission-ai/openspec@latest
cd your-project
openspec init
```

初始化时选择 Claude Code，会在项目中生成 `.claude/commands/openspec/` 目录，包含三个斜杠命令。

#### Spec Miner：代码逆向工程工具（从旧代码中提取规范）

Spec Miner 是 claude-skills 生态中的一个 **Skill**，专为理解复杂、无文档或遗留系统设计。它以“资深软件考古学家”的身份运行，采用**双视角策略**（架构帽分析结构 + QA 帽分析行为）来解码现有实现。

**无需单独安装**，通过 claude-skills 插件即可使用。当你对 Claude 说出触发短语（如“提取这个组件的规范”）时，Skill 自动激活。

#### 两者的区别

| 维度         | OpenSpec                         | Spec Miner                   |
| :----------- | :------------------------------- | :--------------------------- |
| **方向**     | 正向：从规范 → 代码              | 逆向：从代码 → 规范          |
| **适用场景** | 新功能开发、有计划的重构         | 理解遗留代码、接手无文档模块 |
| **使用方式** | 斜杠命令（`/openspec:xxx`）      | 自然语言触发                 |
| **产出**     | proposal.md、design.md、tasks.md | 逆向生成的 spec.md           |
| **安装**     | 需要全局安装 npm 包              | 通过 claude-skills 插件      |


### 二、核心指令速查表

#### OpenSpec 命令（1.2.0 版本后）

| 命令                 | 作用                  | 使用时机                     |
| :------------------- | :-------------------- | :--------------------------- |
| `/openspec:proposal` | 创建变更提案          | 开发新功能、计划重构前       |
| `/openspec:apply`    | 执行已批准的变更      | 提案审查通过后，开始写代码   |
| `/openspec:archive`  | 归档已完成的变更      | 功能上线、代码合并后         |
| `/openspec:explore`  | 探索性分析（v1.2.0+） | 不确定怎么改时，先让 AI 分析 |



#### Spec Miner 触发短语

不需要记斜杠命令，用自然语言描述即可触发：

- `“提取这个模块的规范”` / `“Extract the specification for this component”`
- `“这个 API 的输入输出是什么？”` / `“What are the input/output contracts for this module?”`
- `“从这段代码生成规格文档”` / `“Generate a spec document from this codebase”`
- `“分析这个文件的行为”`

Spec Miner 有两种分析模式：
- **提取模式（extract）**：从代码中自动提取接口、校验规则、数据流
- **分析模式（analyze）**：理解规范之间的关系和系统结构


### 三、两者如何搭配使用

核心原则很简单：

> **Spec Miner 负责“搞清楚现状”，OpenSpec 负责“规划怎么改”。**

| 场景         | 第一步（Spec Miner）               | 第二步（OpenSpec）                            |
| :----------- | :--------------------------------- | :-------------------------------------------- |
| 修复 Bug     | 逆向分析出问题的模块，理解当前行为 | 创建提案描述修复方案，再实施                  |
| 重构遗留模块 | 提取模块的现有行为作为“契约”       | 创建提案规划新架构，按 tasks 逐步迁移         |
| 理解旧代码   | 生成该模块的规格摘要               | 如需修改，再走提案流程                        |
| 新功能开发   | 跳过（没有旧代码可分析）           | 直接 `/openspec:proposal` → `/openspec:apply` |


### 四、四大场景完整工作流

#### 场景 1：修复 Bug

**场景描述**：用户反馈点击登录按钮没反应，你接手排查。

```bash
# Step 1: 先用 Spec Miner 逆向分析出问题的模块
> 提取 src/features/auth/authService.ts 的规范，告诉我当前 login 函数的实际行为是什么。
```

Spec Miner 会分析代码并输出当前的实际行为摘要（如“login 会发送 POST 请求，但错误被 catch 吞掉”）。

```bash
# Step 2: 确认根因后，用 OpenSpec 创建修复提案
> /openspec:proposal 修复登录失败时 UI 无反馈的问题。
> 根因：authService.login 的 catch 块未将错误传递给调用方。
> 修复方案：修改 LoginForm.tsx 的 catch 块，将错误信息设置到 UI 状态。
```

OpenSpec 会生成 `proposal.md` 和 `tasks.md`。

```bash
# Step 3: 审查提案内容，确认后执行
> /openspec:apply fix-login-error-feedback
```

```bash
# Step 4: 修复完成，运行自检
> /pr-prep
```

```bash
# Step 5: 合并后归档
> /openspec:archive fix-login-error-feedback
```


#### 场景 2：重构遗留模块

**场景描述**：`OrderHelper.ts` 有 3000 行，需要拆分但绝不能改崩。

```bash
# Step 1: 用 Spec Miner 锁定现有行为（这是重构的安全网）
> 提取 src/legacy/OrderHelper.ts 的完整规范，包括所有导出函数的输入输出和副作用。
```

Spec Miner 生成一份详细的 `OrderHelper_spec.md`，记录每个函数的签名和行为。

```bash
# Step 2: 基于提取的规范，创建重构提案
> /openspec:proposal 重构 OrderHelper。
> 目标：拆分为 PriceCalculator、Validator、Formatter 三个独立模块。
> 约束：对外暴露的 index.ts 必须保持完全兼容，所有现有调用方不能受影响。
```

OpenSpec 生成 `proposal.md`、`design.md` 和 `tasks.md`，任务被拆成可执行的小步骤。

```bash
# Step 3: 逐步执行，每完成一个 task 就运行测试验证
> /openspec:apply refactor-order-helper
```

```bash
# Step 4: 归档
> /openspec:archive refactor-order-helper
```


#### 场景 3：开发新功能

**场景描述**：需要新增用户头像上传功能，从零开始。

```bash
# Step 1: 直接用 OpenSpec 创建提案（没有旧代码，跳过 Spec Miner）
> /openspec:proposal 实现用户头像上传功能。
> 要求：支持 JPG/PNG，大小不超过 5MB，上传后自动裁剪为 200x200 和 50x50，存储到 OSS。
```

OpenSpec 会追问细节（如“OSS 配置在哪里？”、“裁剪使用什么库？”），最终生成 `proposal.md`、`design.md`、`tasks.md`。

```bash
# Step 2: 审查后执行
> /openspec:apply avatar-upload
```

```bash
# Step 3: 归档
> /openspec:archive avatar-upload
```


#### 场景 4：探索理解旧代码（新人上手）

**场景描述**：突然要修改 `NotificationModule`，但你完全不了解它。

```bash
# Step 1: 用 Spec Miner 快速生成模块摘要
> 用 QA 模式分析 src/modules/notification，告诉我它做什么、有哪些副作用。
```

Spec Miner 输出：“该模块通过 SQS 监听事件，调用 TemplateEngine 渲染后交给 SesClient 发邮件。副作用是在 sent_logs 表写记录。”

```bash
# Step 2: 追问细节
> 如果写日志失败，邮件还会发出去吗？
```

Claude 分析代码流后回答：“不会，代码是先写日志后发邮件，写日志失败会抛异常，导致邮件漏发。”

```bash
# Step 3: 如果这个行为是 Bug，再走场景 1 的流程修复
```


### 五、快速上手指南（5 分钟）

#### 1. 确认工具已安装

```bash
# 检查 OpenSpec
openspec --version

# 检查 Spec Miner（在 Claude Code 中）
> 列出可用的 skills
```

#### 2. 第一个实战：用 Spec Miner 理解一个模块

随便找一个你不太熟悉的 `.ts` 文件：

```bash
> 提取 src/xxx/yyy.ts 的规范，用 QA 模式告诉我它做了什么。
```

看到输出后，你会对 Spec Miner 的能力有直观感受。

#### 3. 第二个实战：用 OpenSpec 修复一个简单 Bug

找一个已知的简单 Bug：

```bash
> /openspec:proposal 修复 [Bug 描述]
```

OpenSpec 会生成提案，审查后执行：

```bash
> /openspec:apply [提案名]
```


### 六、常见问题

**Q：什么时候用 Spec Miner，什么时候用 OpenSpec？**

记住口诀：**旧代码 → Spec Miner；新代码 / 要改动 → OpenSpec**。

**Q：两者能在一个任务里同时用吗？**

可以，而且这是标准做法。修复 Bug 或重构时，先用 Spec Miner 搞清楚现状，再用 OpenSpec 规划改动。

**Q：Spec Miner 生成的规范准确吗？**

Spec Miner 提取的是代码的**实际行为**，不是设计意图。它可能存在遗漏（比如隐式的业务规则在代码里体现不出来），所以**必须人工审查**。这也是为什么 Spec Miner 的文档特别强调“不要假设提取的规范是完整的”。

**Q：OpenSpec 的命令记不住怎么办？**

OpenSpec 初始化后会在 `.claude/commands/` 下生成命令文件，输入 `/openspec:` 然后按 Tab 即可自动补全。


### 七、小结

| 你要做的事                               | 用哪个工具            | 一句话操作                          |
| :--------------------------------------- | :-------------------- | :---------------------------------- |
| 接手一个没文档的模块，想知道它是干什么的 | Spec Miner            | `提取 [模块路径] 的规范`            |
| 修复一个 Bug                             | Spec Miner → OpenSpec | 先逆向分析，再 `/openspec:proposal` |
| 开发一个新功能                           | OpenSpec              | `/openspec:proposal [需求描述]`     |
| 重构一个模块                             | Spec Miner → OpenSpec | 先提取现有规范做安全网，再提案重构  |
| 功能上线后归档                           | OpenSpec              | `/openspec:archive [提案名]`        |
