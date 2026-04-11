---
name: spec-miner-openspec
description: Extract specs from legacy code using Spec Miner and automatically integrate into OpenSpec structure. Use when user wants to reverse-engineer undocumented code and save to openspec/specs.
---

# Spec Miner + OpenSpec 自动集成工具

本 Skill 封装了原生 Spec Miner 的能力，并确保所有生成的规范文档自动归入 OpenSpec 的 `openspec/specs/` 目录，实现单一事实来源。

## 中文触发短语
你可以用以下任一方式呼叫我：
- "用 spec-miner-openspec 分析 [模块路径]"
- "把这个模块的规范提取到 OpenSpec：[文件或目录]"
- "逆向生成规范并保存到 openspec：[目标]"

## 执行流程

### 第一步：调用原生 Spec Miner
对用户指定的目标（文件或目录）执行完整的逆向分析，提取以下信息：
- 公开 API 签名
- 行为契约（输入输出、副作用）
- 错误处理模式
- 关键依赖

> 注意：原生 Spec Miner 默认会将产物输出到临时的 `specs/` 目录。

### 第二步：确定模块名称
根据用户提供的路径，提取一个简洁的模块名：
- 如果是文件：取文件名（不含扩展名），如 `authService.ts` → `authService`
- 如果是目录：取最后一级目录名，如 `src/modules/auth` → `auth`
- 如果用户明确指定了名称，则使用用户指定的名称。

### 第三步：将产物迁移至 OpenSpec 目录
在 Spec Miner 运行结束后，**在同一个 Shell 会话中**执行以下移动操作：

```bash
# 定义路径变量
MODULE_NAME="<这里填入第二步确定的模块名>"
OPENSPEC_TARGET="openspec/specs/$MODULE_NAME"
SPEC_MINER_OUTPUT_DIR="specs"

# 确保目标目录存在
mkdir -p "$OPENSPEC_TARGET"

# 将规范文件移动并重命名为 spec.md
if [ -f "$SPEC_MINER_OUTPUT_DIR"/*_reverse_spec.md ]; then
    mv "$SPEC_MINER_OUTPUT_DIR"/*_reverse_spec.md "$OPENSPEC_TARGET/spec.md"
elif [ -f "$SPEC_MINER_OUTPUT_DIR/spec.md" ]; then
    mv "$SPEC_MINER_OUTPUT_DIR/spec.md" "$OPENSPEC_TARGET/spec.md"
else
    # 兜底：移动第一个找到的 .md 文件
    for file in "$SPEC_MINER_OUTPUT_DIR"/*.md; do
        mv "$file" "$OPENSPEC_TARGET/spec.md"
        break
    done
fi

# 清理临时的 specs 目录（可选）
rm -rf "$SPEC_MINER_OUTPUT_DIR"

echo "✅ 规范文档已保存至：$OPENSPEC_TARGET/spec.md"
```

### 第四步：向用户报告结果
用中文告知用户最终生成的文件位置，并提示后续可进行的操作，例如：
- 查看或编辑生成的 `spec.md`
- 基于此规范创建 OpenSpec 变更提案：`/openspec:proposal`

## 使用示例

**用户输入：**
> 用 spec-miner-openspec 分析 src/modules/payment

**AI 执行过程：**
1. 调用 Spec Miner 分析 `src/modules/payment` 目录。
2. 原生输出临时文件至 `specs/`。
3. 提取模块名 `payment`。
4. 执行迁移脚本，将文件移至 `openspec/specs/payment/spec.md`。
5. 报告："✅ 规范文档已保存至：openspec/specs/payment/spec.md"

## 注意事项
- 如果 `openspec/specs/` 目录下已存在同名模块的 `spec.md`，本次操作会**覆盖**原文件。如需保留历史版本，请提前备份。
- 生成的规范是基于代码**实际行为**的逆向推导，不保证完全准确，请务必人工审查后再作为开发依据。
```

### 关键设计说明

| 部分 | 语言选择 | 原因 |
| :--- | :--- | :--- |
| `name` 和 `description` | **英文** | Claude Code 的 Skill 匹配引擎对英文语义理解更稳定，这是官方推荐做法。 |
| 触发短语 | **中英双语** | 提供中文触发短语示例，方便您自然使用，AI 同样能识别。 |
| 执行指令和脚本 | **英文 / Bash** | 脚本命令本身是英文环境，保持原样即可。 |
| 注释和说明 | **中文** | 方便您和团队阅读维护。 |

现在您只需将此文件保存到 `.claude/skills/spec-miner-openspec/SKILL.md`，然后在对话中说 **“用 spec-miner-openspec 分析 src/xxx”** 即可触发自动集成流程。