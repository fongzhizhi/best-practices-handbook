> CreateDate: 2026-04-09

# Claude Code - 工具推荐

## 一、前置环境（必需）

### 1. Node.js（v18 或更高版本）
ClawHub、OpenSpec 等工具都依赖 Node.js 环境。

```bash
# 验证是否已安装
node -v
npm -v

# 如未安装，从官网下载安装包：https://nodejs.org/
```

### 2. Python（3.11 或更高版本）—— 仅当使用 GitHub Spec Kit 时需要
GitHub Spec Kit 基于 Python 构建，需要 Python 环境。  
**注意**：在内网离线环境中，Python 依赖树复杂，安装可能较为繁琐。如果你的内网环境难以满足 Python 依赖，建议直接选用下方的 OpenSpec（纯 Node.js 方案）。

```bash
# 验证是否已安装
python --version

# 如未安装，从官网下载安装包：https://www.python.org/
```


## 二、核心工具安装（按推荐顺序）

### 1. GitHub Spec Kit（新需求开发 + 重构规划）—— 功能完整，但依赖 Python 生态

Spec Kit 是 GitHub 开源的规范驱动开发工具包，提供 `/speckit.specify`、`/speckit.plan` 等命令，流程严谨，适合从零开始的新项目。

**优点**：流程完整，社区活跃，文档丰富。  
**缺点**：依赖 Python + `uv`，在内网离线环境中安装较为复杂（需处理多层依赖）。

**推荐安装方式（持久化安装）**：

```bash
# 使用 uv 工具安装（持久化安装，推荐）
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

**或使用 Snap 安装（Linux）**：

```bash
sudo snap install spec-kit
```

**验证安装**：

```bash
specify --version
```

**在 Claude Code 中使用**：Spec Kit 提供了 MCP Server 版本，可以与 Claude Code 集成。配置方式稍后说明。

> **内网离线建议**：如果 Python 依赖问题难以解决，请直接使用下方的 **OpenSpec** 方案。

---

### 2. OpenSpec（新需求开发 + Bug修复 + 重构规划）—— 轻量级，纯 Node.js，内网友好

OpenSpec 是一个专为现有项目（“棕地”项目）设计的轻量级规范驱动开发框架。它基于 Node.js，无需 Python 环境，安装极其简单，非常适合内网离线环境。

**核心命令**：`/openspec:proposal`、`/openspec:apply`、`/openspec:archive`。

**优点**：
- 纯 Node.js，与 TypeScript 项目技术栈一致。
- 安装简单：一条 `npm install -g` 即可。
- 内网离线友好：可提前下载 `.tgz` 包离线安装。
- 变更隔离设计，适合大型多人协作项目。

**安装方式**：

```bash
# 全局安装（在线环境）
npm install -g @fission-ai/openspec@latest

# 内网离线安装：在外网下载 .tgz 包，拷贝到内网执行
# 1. 在外网执行：npm pack @fission-ai/openspec
# 2. 将生成的 .tgz 文件拷贝到内网
# 3. 在内网执行：npm install -g ./openspec-*.tgz
```

**验证安装**：

```bash
openspec --version
```

**在 Claude Code 中使用**：在项目根目录运行 `openspec init`，会自动生成 `.claude/commands/openspec-*.md` 斜杠命令文件。之后在 Claude Code 中即可使用 `/openspec:proposal`、`/openspec:apply`、`/openspec:archive`。

> **对比总结**：如果你希望使用功能最完整的规范驱动流程，且能解决 Python 依赖问题，选择 **GitHub Spec Kit**；如果你追求轻量、快速落地、内网环境简单可靠，**OpenSpec** 是更优选择。

---

### 3. OpenClaw Skills 生态（含 ClawHub + Spec Miner）

OpenClaw 是一个 AI 代理平台，其 Skills 生态包含了 Spec Miner 等技能。

#### 3.1 安装 ClawHub CLI（技能包管理器）

ClawHub 是 OpenClaw 生态的公共技能注册中心，用于安装和管理 AI 技能包。

```bash
# 全局安装 ClawHub CLI
npm i -g clawhub

# 验证安装
clawhub --version
```

#### 3.2 安装 Spec Miner 技能

Spec Miner 是 OpenClaw Skills 库中的专业级工具，专为理解复杂、缺乏文档的遗留系统设计。

```bash
# 通过 ClawHub 安装 spec-miner
clawhub install spec-miner
```

**或使用 npx 直接安装**：

```bash
npx clawhub@latest install spec-miner
```

安装后，Spec Miner 会被放置到 `~/.openclaw/skills/` 目录下。

**内网离线安装 Spec Miner 的替代方法**：
1. 在外网下载 `spec-miner` 技能压缩包（从 GitHub Releases 或 OpenClaw Skills 仓库获取）。
2. 拷贝到内网，解压到 `~/.openclaw/skills/` 目录。
3. 在 Claude Code 中运行 `openclaw skills reload` 即可。

#### 3.3 （可选）安装 OpenClaw CLI

如果需要更完整的 OpenClaw 管理能力，可以安装 OpenClaw CLI：

```bash
npm install -g openclaw@latest
```

**验证安装**：

```bash
openclaw --version
openclaw skills check
```


## 三、可选工具（按需安装）

### 1. MCP Server 支持（仅 Spec Kit 需要）

如果希望 Claude Code 通过 MCP 与 Spec Kit 深度集成，可以配置 Spec Kit MCP Server。

```bash
npm install -g @modelcontextprotocol/spec-kit-server
```

### 2. Drift（文档与代码一致性检测）

Drift 是一个检测 Spec 文档与代码是否一致的工具，建议在 CI 中使用。

```bash
npm install -g @drift/cli
```

### 3. Claude-mem（上下文记忆工具）

如果需要 AI 跨会话记住项目上下文，可以安装 claude-mem：

```bash
npm install -g claude-mem
```


## 四、各仓库的配置文件

工具安装完成后，需要在**每个子仓库**的 `CLAUDE.md` 中引用宪法和工作流规则：

```markdown
# 项目核心规则
请严格遵守我们的项目宪法：
@.claude/rules/constitution.md
@.claude/rules/workflow-router.md
```


## 五、工具清单汇总

| 工具                         | 用途                              | 安装命令                                                                 | 优先级   | 内网离线难度 |
| ---------------------------- | --------------------------------- | ------------------------------------------------------------------------ | -------- | ------------ |
| **Node.js v18+**             | 运行环境（前置依赖）              | 官网下载                                                                 | **必需** | 低           |
| **Python 3.11+**             | Spec Kit 所需环境（仅 Spec Kit）  | 官网下载                                                                 | 可选     | 中           |
| **GitHub Spec Kit**          | 新需求 + 重构规划（完整流程）     | `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git` | 核心     | **高**       |
| **OpenSpec**                 | 新需求 + Bug + 重构（轻量流程）   | `npm install -g @fission-ai/openspec@latest`                            | **核心** | **低**       |
| **ClawHub CLI**              | 技能包管理器                      | `npm i -g clawhub`                                                       | 核心     | 低           |
| **Spec Miner**               | 逆向分析遗留代码                  | `clawhub install spec-miner`                                             | 核心     | 低           |
| **OpenClaw CLI**             | OpenClaw 平台管理（可选）         | `npm install -g openclaw@latest`                                         | 可选     | 低           |
| **Spec Kit MCP Server**      | Spec Kit 与 Claude Code 集成      | `npm install -g @modelcontextprotocol/spec-kit-server`                   | 可选     | 中           |
| **Drift**                    | 文档与代码一致性检测              | `npm install -g @drift/cli`                                              | 可选     | 低           |
| **Claude-mem**               | 跨会话上下文记忆                  | `npm install -g claude-mem`                                              | 可选     | 低           |


## 六、安装后的验证检查清单

```bash
# 1. 验证 Node.js 环境
node -v  # 应显示 v18 或更高

# 2. 验证 Python 环境（仅当使用 Spec Kit 时需要）
python --version  # 应显示 3.11 或更高

# 3. 验证 OpenSpec（如果选择）
openspec --version

# 4. 验证 Spec Kit（如果选择）
specify --version

# 5. 验证 ClawHub
clawhub --version

# 6. 验证 Spec Miner（列出已安装技能）
clawhub list
# 或检查技能目录
ls ~/.openclaw/skills/  # macOS/Linux
dir %USERPROFILE%\.openclaw\skills\  # Windows

# 7. 验证 Claude Code 能正确加载配置
claude --version
```


## 七、常见问题与排查

### Q1：ClawHub 安装失败？
确保 Node.js 版本 ≥ v18，并且 npm 网络正常。可以尝试使用淘宝镜像：

```bash
npm config set registry https://registry.npmmirror.com
npm i -g clawhub
```

### Q2：Spec Miner 安装后无法使用？
检查技能是否正确安装到 `~/.openclaw/skills/spec-miner/` 目录。如果目录为空，尝试重新安装：

```bash
clawhub uninstall spec-miner
clawhub install spec-miner
```

### Q3：Spec Kit 命令找不到？
如果使用 `uv tool install` 安装，确保 `uv` 工具的 bin 目录在 PATH 中：

```bash
uv tool update-shell  # 更新 shell 配置
# 或手动添加路径（Windows 示例）
set PATH=%USERPROFILE%\.local\bin;%PATH%
```

### Q4：在内网安装 OpenSpec 时如何离线？
```bash
# 在外网有网络的环境执行
npm pack @fission-ai/openspec
# 会生成 openspec-*.tgz 文件，拷贝到内网
# 在内网执行
npm install -g ./openspec-*.tgz
```

### Q5：GitHub Spec Kit 和 OpenSpec 应该如何选择？

| 对比维度       | GitHub Spec Kit                  | OpenSpec                         |
| -------------- | -------------------------------- | -------------------------------- |
| **适用场景**   | 新项目、完整规范流程             | 现有项目、轻量迭代               |
| **依赖环境**   | Python + uv（复杂）              | Node.js（简单）                  |
| **内网安装难度** | 高（需解决多层依赖）             | 低（npm pack 离线即可）          |
| **工作流长度** | 长（6+ 阶段）                    | 短（propose → apply → archive）  |
| **与 Claude Code 集成** | 需配置 MCP Server          | 原生支持，自动生成斜杠命令       |
| **推荐场景**   | 团队有严格规范要求，能解决 Python 环境 | 团队希望快速落地，内网环境受限   |

### Q6：多个工具如何协同工作？
- **Claude Code** 是主控 AI，负责与用户交互。
- **OpenSpec** 或 **Spec Kit** 提供规范驱动工作流，任选其一即可（不建议同时使用，以免流程冲突）。
- **Spec Miner** 作为 OpenClaw Skill，用于逆向分析遗留代码，可与上述任一流程搭配使用。
```

主要修改点：
1. 在“前置环境”中标注 Python 仅 Spec Kit 需要，并说明内网安装复杂。
2. 在“核心工具安装”中保留 Spec Kit，增加内网注意事项，并新增 OpenSpec 章节，详细说明其优点、安装方式、内网离线方法。
3. 调整章节顺序：Spec Kit → OpenSpec → OpenClaw Skills。
4. 在工具清单表格中增加 OpenSpec 行，增加“内网离线难度”列。
5. 在验证清单中增加 OpenSpec 验证。
6. 在常见问题中增加 Q4（OpenSpec 离线安装）和 Q5（选择对比）。