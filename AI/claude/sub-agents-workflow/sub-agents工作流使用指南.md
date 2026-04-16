---
createDate: 2026-04-15
---

# Sub-agents 工作流使用指南

改造背景和更多思路可参考文档：《从 Commands/Skills 到 Subagents 的架构升级》

## 一、架构设计概览

我们将原有的“智能指令”体系升级为**三层分离的 Agent 协作架构**，实现“调度智能、执行纯粹、能力复用”。

```
┌─────────────────────────────────────────────────┐
│                 用户（你）                        │
└────────────────────┬────────────────────────────┘
                     │ 自然语言描述任务
                     ▼
┌─────────────────────────────────────────────────┐
│          调度层：Orchestrator Agent               │
│   - 识别意图（新功能/Bug/重构/分析）              │
│   - 拆解任务、补全信息（ONES、分支等）            │
│   - 调度专业 Agent，汇总结果                      │
└────────────────────┬────────────────────────────┘
                     │ 调用 Task 工具
        ┌────────────┼────────────┐
        ▼            ▼            ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  analyzer    │ │ spec-writer  │ │ implementer  │
│  代码分析专家 │ │  规范撰写专家 │ │  实施交付专家 │
└──────────────┘ └──────────────┘ └──────────────┘
        │               │               │
        └───────────────┼───────────────┘
                        │ 调用原子指令 / Skills
                        ▼
┌─────────────────────────────────────────────────┐
│         原子能力层：Commands + Skills             │
│  /git:* 极简 Git 命令 | /quality:* 质量检查       │
│  spec-miner | ones-parser | gitlab-config       │
└─────────────────────────────────────────────────┘
```

**核心理念**：

- **你只负责描述任务**，Orchestrator 自动判断该走什么流程。
- **Agent 各司其职**，分析、写规范、编码、提交都由专业 Agent 完成。
- **执行速度大幅提升**，原子指令不再进行多余的“思考”和询问。

---

## 二、目录结构说明

升级后的配置目录如下（位于项目根目录 `.claude/` 下）：

```txt
项目根目录/
├── .claude/
│   ├── agents/                        # 自定义子代理
│   │   ├── orchestrator.md				# 主控调度
│   │   ├── analyzer.md					# 代码分析
│   │   ├── spec-writer.md				# 规范撰写
│   │   └── implementer.md				# 实施交付
│   │
│   ├── commands/                      # 自定义的快捷指令
│   │   ├── opsx/                      # openspec指令包
│   │   │   ├── apply.md
│   │   │   ├── archive.md
│   │   │   ├── explore.md
│   │   │   ├── propose.md
│   │   ├── git/                       # git指令
│   │   │   ├── checkout.md            # 切换分支
│   │   │   ├── branch.md              # 创建分支
│   │   │   ├── commit.md              # 执行提交
│   │   │   ├── push.md                # 推送提交
│   │   │   ├── pull.md                # 拉取代码
│   │   │   ├── fetch.md               # 获取远程信息
│   │   │   ├── rebase.md              # 变基操作
│   │   │   ├── merge.md               # 合并请求
│   │   │   ├── cherry-pick.md         # 遴选合并
│   │   │   ├── stash.md               # 缓存变更
│   │   │   ├── status.md              # 查看状态
│   │   │   ├── diff.md                # 查看差异
│   │   │   └── log.md                 # 查看日志
│   │   ├── quality/                   # 质量保障指令
│   │   │   ├── pre-commit.md          # 预提交检测
│   │   │   └── prettier-format.md     # 变更格式化
│   │
│   ├── skills/                        # 自定义的技能包
│   │   ├── spec-miner/				   # spec-miner技能包
│   │   │   └── SKILL.md
│   │   ├── ones-parser/			   # ones解析
│   │   │   └── SKILL.md
│   │   ├── gitlab-config/			   # gitlab配置解析
│   │   │   └── SKILL.md
│   │   └── constitution/			   # 场景识别, 宪法规范
│   │       └── SKILL.md
│   │   └── openspec-xxx/			  # openspec相关技能包
│   │
│   ├── rules/                         # 规则文件
│   │   ├── constitution.md            # 编码规范
│   │   └── workflow-router.md         # 智能引导规则
│   │
│   └── settings.json                  # 权限配置 + hooks + GitLab用户映射
│
└── CLAUDE.md                          # 项目级入口，引用上述规则
```

**重要变化**：

- Git 相关指令从 `/git-checkout` 变为 `/git:checkout`（目录化调用）。
- 质量指令变为 `/quality:pre-commit`、`/quality:prettier-format`。

---

## 三、准备工作

### 3.1 环境要求

- Claude Code CLI 已安装并登录
- Git ≥ 2.18
- GitLab 版本 ≥ 11.10（如需自动创建 MR）
- Node.js ≥ 20.19.0（如需 OpenSpec CLI）

### 3.2 配置文件检查

确保 `.claude/settings.json` 包含以下配置（按需修改）：

```json
{
  "gitlab": {
    "token": "你的 GitLab Personal Access Token",
    "host": "https://gitlab.your-company.com",
    "userIds": {
      "alice": 123,
      "bob": 456
    }
  },
  "mr": {
    "defaultTargetBranch": "develop",
    "defaultReviewers": ["alice", "bob"]
  },
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Bash(curl:*)"
    ]
  }
}
```

**字段说明**：

| 字段                     | 用途                                          | 对应 Skill/Agent               |
| :----------------------- | :-------------------------------------------- | :----------------------------- |
| `gitlab.token`           | GitLab Personal Access Token（需 `api` 权限） | `gitlab-config`                |
| `gitlab.host`            | GitLab 服务器地址                             | `gitlab-config`                |
| `gitlab.userIds`         | 用户名到数字 ID 的本地映射（可选）            | `gitlab-config`                |
| `mr.defaultTargetBranch` | MR 默认目标分支                               | `gitlab-config`、`implementer` |
| `mr.defaultReviewers`    | 默认审核人（用户名或 ID 数组）                | `gitlab-config`、`implementer` |
| `permissions.allow`      | 允许 Claude Code 自动执行的命令               | 全局生效                       |

> **注意**：Token 获取方式：登录 GitLab → Settings → Access Tokens，勾选 `api` 和 `write_repository` 权限。

### 3.3 激活 Orchestrator

在 `CLAUDE.md` 中添加以下引用，使 Orchestrator 成为默认入口：

```markdown
## AI 工作流入口

@.claude/agents/orchestrator.md

## 编码规范

@.claude/rules/constitution.md
```

重启 Claude Code 会话即可生效。

---

## 四、典型使用案例

以下场景中，你只需用自然语言描述任务，Orchestrator 会自动调度合适的 Agent 完成。

### 案例一：开发新功能

**你的输入**：
> “我要开发一个用户头像上传功能，支持 JPG/PNG，大小不超过 5MB。ONES 单号是 PRO-151081。”

**AI 自动执行流程**：

1. Orchestrator 识别为“新功能开发”。
2. 调用 `ones-parser` 获取 ONES 单详情。
3. 调度 `spec-writer` 生成 OpenSpec 提案：
   - 创建 `openspec/changes/feat/PRO-151081-avatar-upload/`
   - 生成 `proposal.md` 和 `tasks.md`
4. 向你展示提案摘要，询问是否继续。
5. 你确认后，调度 `implementer`：
   - 创建分支 `feat/PRO-151081-avatar-upload`
   - 按 tasks.md 实施编码
   - 格式化、质量检查、提交、推送
   - 调用 `gitlab-config` 获取目标分支和审核人，创建 GitLab MR
6. 返回 MR 链接，提示可归档规范。

**你只需说“继续”或“确认”，其余全自动完成。**

---

### 案例二：修复 Bug（含逆向分析）

**你的输入**：
> “订单金额计算有误，用了优惠券后多扣了 10 块钱。相关代码在 `src/services/order.ts`，ONES 单 PRO-67890。”

**AI 自动执行流程**：

1. Orchestrator 识别为“Bug 修复”。
2. 调度 `analyzer` 分析 `src/services/order.ts`：
   - 调用 `spec-miner` Skill 提取当前行为规范
   - 定位根因（如折扣计算函数未处理边界）
   - 返回结构化分析报告
3. 调度 `spec-writer` 基于分析报告创建修复提案：
   - `openspec/changes/fix/PRO-67890-order-discount-fix/`
4. 调度 `implementer` 实施修复、提交、创建 MR。
5. 返回 MR 链接。

**连问题定位带修复一气呵成，无需手动分析代码。**

---

### 案例三：重构遗留代码

**你的输入**：
> “`src/legacy/helper.ts` 这个文件太臃肿了，我想把它拆分成三个独立模块。”

**AI 自动执行流程**：

1. Orchestrator 识别为“代码重构”。
2. 调度 `analyzer` 分析 `helper.ts`，生成现状规范摘要。
3. 调度 `spec-writer` 创建重构提案，包含拆分方案和迁移步骤。
4. 你审阅提案后确认。
5. 调度 `implementer` 执行拆分、提交、创建 MR。

**重构有规范保驾护航，避免改坏原有逻辑。**

---

### 案例四：仅分析代码（不产生变更）

**你的输入**：
> “帮我梳理一下 `src/auth/login.ts` 的登录流程，我想理解它的逻辑。”

**AI 自动执行流程**：

1. Orchestrator 识别为“代码分析”。
2. 调度 `analyzer` 分析目标文件。
3. 返回结构化分析报告（模块职责、核心流程、依赖关系）。
4. **不会产生任何文件变更或 Git 操作。**

---

### 案例五：仅提交已完成的代码

**你的输入**：
> “我代码改完了，帮我提交并创建 MR，目标分支是 develop，审核人 @alice。”

**AI 自动执行流程**：

1. Orchestrator 识别为“代码交付”。
2. 调度 `implementer`：
   - 检测当前变更，调用 `ones-parser` 和 `gitlab-config` 生成符合规范的 commit message
   - 执行 `/git:commit`、`/git:push`
   - 调用 `gitlab-config` 获取审核人 ID，创建 MR
3. 返回 MR 链接。

**你只管写代码，提交合并 AI 全包。**

---

### 案例六：手动调用原子指令（高级用户）

如果你想跳过调度层，直接使用极简 Git 指令（速度更快）：

| 旧指令                 | 新指令                            |
| :--------------------- | :-------------------------------- |
| `/git-checkout main`   | `/git:checkout main`              |
| `/git-branch feat/xxx` | `/git:branch feat/xxx`            |
| `/git-commit "msg"`    | `/git:commit "msg"`               |
| `/git-push`            | `/git:push origin HEAD`           |
| `/prettier-format`     | `/quality:prettier-format <文件>` |

这些指令**不做任何状态检测和询问**，直接执行，适合熟悉 Git 的开发者快速操作。

---

## 五、常见问题

**Q：我不想用 Orchestrator，还能用以前的方式吗？**  
A：可以。你可以直接调用原子指令（如 `/git:commit`），或直接指定某个 Agent（如“用 analyzer 分析这个文件”）。Orchestrator 是推荐入口，但不强制。

**Q：Agent 执行过程中我想中断怎么办？**  
A：直接在对话中说“停止”或按 `Ctrl+C`，Orchestrator 会暂停并询问下一步。

**Q：如果 GitLab Token 没配置好，MR 创建失败怎么办？**  
A：Orchestrator 会提示错误，并引导你检查 `.claude/settings.json` 配置。你也可以手动创建 MR。

**Q：为什么 Git 指令变快了？**  
A：因为我们把指令中的“决策逻辑”（如检测未提交代码、弹选项）全部移除，交给上层 Agent 处理。原子指令只做执行，速度从分钟级降到秒级。

**Q：`settings.json` 中的 `reviewers` 和 `userIds` 有什么区别？**  
A：`mr.defaultReviewers` 是默认审核人列表（支持用户名或 ID），`gitlab.userIds` 是用户名到数字 ID 的映射表，用于将用户名转换为 API 所需的数字 ID。两者配合使用，`gitlab-config` Skill 会自动处理转换。

---

## 六、反馈与支持

使用过程中如有问题或改进建议，请联系架构组或在项目仓库提交 Issue。我们持续优化这套工作流，让它更贴合团队的实际开发习惯。