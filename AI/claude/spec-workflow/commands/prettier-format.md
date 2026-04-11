请对当前分支中所有已变更的代码文件执行 Prettier 格式化。

## 执行步骤

### 1. 获取变更文件列表
执行 `git diff --name-only` 获取工作区变更文件。
执行 `git diff --cached --name-only` 获取暂存区变更文件。
合并去重，筛选出需格式化的文件类型：`.ts`, `.tsx`, `.js`, `.jsx`, `.json`, `.md`, `.css`, `.scss`, `.html`。

### 2. 检测并使用项目格式化命令
读取项目根目录的 `package.json`，检查 `scripts` 字段：
- 若存在 `format` 或 `prettier`相关的脚本，优先执行 `npm run format -- <文件列表>`（或 `yarn format`）。
- 若存在 `prettier` 或 `prettier:diff` 等明确指向 Prettier 的脚本，执行对应命令。
- 若脚本不支持传入文件参数，则对每个文件单独执行格式化命令。

### 3. 若无格式化命令，执行 Prettier 并提示配置
- 检查 `node_modules/.bin/prettier` 是否存在，若不存在则执行 `npx prettier --version` 检测。
- 对变更文件列表执行 `npx prettier --write <文件1> <文件2> ...`。
- 格式化完成后，提示用户：

 ```txt
  ⚠️ 项目 package.json 中未配置格式化脚本。
 ```

  建议添加以下脚本以便统一使用：
 ```json
 "scripts": {
     "format": "prettier --write \"src/**/*.{ts,tsx,js,jsx,json,md}\"",
     "format:diff": "git diff HEAD --name-only --diff-filter=d | xargs format --write"
  }
 ```

  配置后即可使用 `npm run format` 或 `/prettier-format` 调用。

### 4. 输出结果

✅ 已格式化 X 个文件。

变更文件列表：

- src/components/Avatar.tsx
- src/utils/format.ts
## 注意事项
- 仅格式化已变更的文件，避免触碰无关代码，减少 PR 噪音。
- 若格式化后产生大量变更（超过 10 个文件或 200 行），提示用户：

  ```txt
  ⚠️ 本次格式化影响范围较大，建议将其作为独立 commit 提交。
  ```

