# Claude Code工具内网安装思路：OpenSpec + Spec Miner案例

## 一、核心原理

Claude Code 的所有扩展能力（斜杠命令、技能包、规则文件）都只是 **Markdown 文本文件**，存放在项目的 `.claude/` 目录下。只要把这份配置复制到任何项目根目录，Claude Code 启动时会自动加载，**不需要在内网运行任何安装命令**。

> **为什么之前教程那么复杂？**  
> 因为最初选择了 GitHub Spec Kit，它依赖 Python 环境和动态脚本生成，无法仅靠复制文件完成。而 **OpenSpec + Spec Miner** 是纯 Markdown 配置，完美支持“复制即用”。

## 二、外网准备（只需做一次）

找一台能联网的电脑，创建一个临时目录，执行以下操作：

### 1. 安装 OpenSpec（仅用于生成配置文件）

```bash
npm install -g @fission-ai/openspec@latest
mkdir my-template && cd my-template
openspec init --tools claude
```

这一步会在当前目录生成 `.claude/commands/openspec/` 三个命令文件：
- `proposal.md`
- `apply.md`
- `archive.md`

### 2. 下载 Spec Miner 技能包

```bash
# 克隆 OpenClaw Skills 仓库（或直接下载 ZIP）
git clone https://github.com/openclaw/openclaw-skills.git
# 将 spec-miner 技能文件夹复制到 .claude/skills/
cp -r openclaw-skills/skills/spec-miner .claude/skills/
```

### 3. 添加项目级规则（可选但推荐）

创建 `.claude/rules/constitution.md`（项目宪法）和 `.claude/rules/workflow-router.md`（AI 引导规则）。  
你可以直接使用之前写好的版本，或者用下面的最小示例：

**`.claude/rules/constitution.md`**（最小示例）：
```markdown
# 项目宪法
- 所有新代码必须通过 TypeScript strict 检查
- 禁止使用 `any`（除非有例外注释）
- 修改核心模块必须更新对应的 spec 文档
```

**`.claude/rules/workflow-router.md`**（最小示例）：
```markdown
# AI 引导规则
- 当用户说“新功能”或“添加xxx” → 引导使用 `/openspec:proposal`
- 当用户说“修复bug” → 先分析问题，再创建提案
- 当用户说“分析xxx模块” → 自动调用 Spec Miner
```

### 4. 创建根目录 `CLAUDE.md`

在 `my-template/` 根目录创建 `CLAUDE.md`，内容：

```markdown
# 项目通用配置

请遵守以下规则：
@.claude/rules/constitution.md
@.claude/rules/workflow-router.md

## 可用技能
- Spec Miner：用于逆向分析遗留代码，输入“用 Spec Miner 分析 xxx”

## 可用命令
- `/openspec:proposal` - 创建变更提案
- `/openspec:apply` - 实施已批准的提案
- `/openspec:archive` - 归档已完成的变更
```

### 5. 打包 `.claude` 目录

```bash
zip -r claude-config.zip .claude CLAUDE.md
```

## 三、内网复制（每个仓库执行一次）

1. 将 `claude-config.zip` 拷贝到内网开发机。
2. 进入你的子仓库根目录（例如 `D:\Code\editor-pro\pro-sch`）。
3. 解压压缩包，确保 `.claude` 目录和 `CLAUDE.md` 出现在根目录。
4. （可选）如果子仓库有特殊配置，可以在 `CLAUDE.md` 中追加仓库专属内容。

## 四、验证

打开 Claude Code（确保工作目录是子仓库根目录），输入：

```
/openspec:proposal
```

如果能正常输出提案模板，说明 OpenSpec 命令生效。  
再输入：

```
用 Spec Miner 分析一下 src/main.ts
```

如果 AI 回复中出现了“架构帽”、“QA帽”或主动调用 `grep` 搜索代码，说明 Spec Miner 技能已激活。

## 五、为什么这样做能工作？

| 组件              | 本质                                          | 是否需要内网安装 |
| ----------------- | --------------------------------------------- | ---------------- |
| OpenSpec 斜杠命令 | `.claude/commands/openspec/*.md` 静态文件     | 否               |
| Spec Miner 技能   | `.claude/skills/spec-miner/SKILL.md` 静态文件 | 否               |
| 宪法与规则        | `.claude/rules/*.md` 静态文件                 | 否               |
| `CLAUDE.md`       | 项目根目录的静态文件                          | 否               |

Claude Code 启动时会**递归扫描**项目根目录下的 `.claude/` 文件夹，自动注册所有命令、技能和规则。整个过程不依赖任何外部工具或网络，因此内网无需安装任何东西。

## 六、维护与更新

- 当需要更新 OpenSpec 命令模板时，在外网重新运行 `openspec init`，然后重新打包 `.claude/commands/openspec/` 目录，复制到内网覆盖。
- 当需要更新 Spec Miner 技能时，拉取最新的 OpenClaw Skills 仓库，替换 `.claude/skills/spec-miner/` 文件夹。
- 当需要修改团队规范时，直接编辑 `.claude/rules/constitution.md`，提交到 Git 仓库，团队成员拉取即可。

---

**总结**：一次外网配置，内网无限复制。这比任何安装脚本都简单可靠。