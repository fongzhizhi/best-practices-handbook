

> CreateDate: 2026-04-09

# Claude Code - 工具推荐

## 一、前置环境（必需）

### 1. Node.js（v18 或更高版本）
ClawHub、Spec Kit 等工具都依赖 Node.js 环境。

```bash
# 验证是否已安装
node -v
npm -v

# 如未安装，从官网下载安装包：https://nodejs.org/
```

### 2. Python（3.11 或更高版本）
Spec Kit CLI 工具是基于 Python 构建的，需要 Python 环境。

```bash
# 验证是否已安装
python --version

# 如未安装，从官网下载安装包：https://www.python.org/
```


## 二、核心工具安装（按推荐顺序）

### 1. GitHub Spec Kit（新需求开发 + 重构规划）

Spec Kit 是 GitHub 开源的规范驱动开发工具包，提供 `/speckit.specify`、`/speckit.plan` 等命令。

**推荐安装方式（持久化安装）** ：

```bash
# 使用 uv 工具安装（持久化安装，推荐）
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

**或使用 Snap 安装（Linux）** ：

```bash
sudo snap install spec-kit
```

**验证安装**：

```bash
specify --version
```

**在 Claude Code 中使用**：Spec Kit 提供了 MCP Server 版本，可以与 Claude Code 集成。配置方式稍后说明。

### 2. OpenClaw Skills 生态（含 ClawHub + Spec Miner）

OpenClaw 是一个 AI 代理平台，其 Skills 生态包含了 Spec Miner 等技能。

#### 2.1 安装 ClawHub CLI（技能包管理器）

ClawHub 是 OpenClaw 生态的公共技能注册中心，用于安装和管理 AI 技能包。

```bash
# 全局安装 ClawHub CLI
npm i -g clawhub

# 验证安装
clawhub --version
```

#### 2.2 安装 Spec Miner 技能

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

#### 2.3 （可选）安装 OpenClaw CLI

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

### 1. MCP Server 支持

如果希望 Claude Code 通过 MCP（Model Context Protocol）与 Spec Kit 深度集成，可以配置 Spec Kit MCP Server。

MCP Server 通常通过 npm 安装：

```bash
npm install -g @modelcontextprotocol/spec-kit-server
```

然后在 Claude Code 的 MCP 配置文件中添加相应配置。

### 2. Drift（文档与代码一致性检测）

Drift 是一个检测 Spec 文档与代码是否一致的工具，建议在 CI 中使用。

```bash
# 通过 npm 安装
npm install -g @drift/cli

# 或通过 cargo 安装（Rust 版本）
cargo install drift-cli
```

### 3. Claude-mem（上下文记忆工具）

如果需要 AI 跨会话记住项目上下文，可以安装 claude-mem：

```bash
npm install -g claude-mem
```


## 四、各仓库的配置文件

工具安装完成后，需要在**每个子仓库**的 `CLAUDE.md` 中引用宪法：

```markdown
# 项目核心规则
请严格遵守我们的项目宪法：
@.claude/rules/constitution.md
```


## 五、工具清单汇总

| 工具                    | 用途                      | 安装命令                                                     | 优先级   |
| ----------------------- | ------------------------- | ------------------------------------------------------------ | -------- |
| **Node.js v18+**        | 运行环境（前置依赖）      | 官网下载                                                     | **必需** |
| **Python 3.11+**        | 运行环境（前置依赖）      | 官网下载                                                     | **必需** |
| **GitHub Spec Kit**     | 新需求 + 重构规划         | `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git` | **核心** |
| **ClawHub CLI**         | 技能包管理器              | `npm i -g clawhub`                                           | **核心** |
| **Spec Miner**          | 逆向分析遗留代码          | `clawhub install spec-miner`                                 | **核心** |
| **OpenClaw CLI**        | OpenClaw 平台管理（可选） | `npm install -g openclaw@latest`                             | 可选     |
| **Spec Kit MCP Server** | 与 Claude Code 集成       | `npm install -g @modelcontextprotocol/spec-kit-server`       | 可选     |
| **Drift**               | 文档与代码一致性检测      | `npm install -g @drift/cli`                                  | 可选     |
| **Claude-mem**          | 跨会话上下文记忆          | `npm install -g claude-mem`                                  | 可选     |


## 六、安装后的验证检查清单

```bash
# 1. 验证 Node.js 环境
node -v  # 应显示 v18 或更高

# 2. 验证 Python 环境
python --version  # 应显示 3.11 或更高

# 3. 验证 Spec Kit
specify --version

# 4. 验证 ClawHub
clawhub --version

# 5. 验证 Spec Miner（列出已安装技能）
clawhub list
# 或检查技能目录
ls ~/.openclaw/skills/  # macOS/Linux
dir %USERPROFILE%\.openclaw\skills\  # Windows

# 6. 验证 Claude Code 能正确加载配置
claude --version
```

---

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

### Q4：多个工具如何协同工作？
- **Claude Code** 是主控 AI，负责与用户交互
- **Spec Kit** 通过 MCP Server 与 Claude Code 集成，提供 `/speckit.*` 命令
- **Spec Miner** 作为 OpenClaw Skill，可通过 `clawhub install spec-miner` 安装，然后在 Claude Code 对话中直接调用（例如"使用 Spec Miner 分析 src/order 模块"）

