# cli-change

## 目的 (Purpose)

定义 `openspec change` 命名空间子命令（`show` / `list` / `validate`）的对外行为——展示 change 提案、列出所有 pending change、校验 change 结构。该 spec 与 `cli-show` 一起承担 verb-noun 迁移中的"老路径兼容"角色。

## 内容解析 (Content Analysis)

### 1. 三大子命令
- **`openspec change show <id>`**
  - 默认渲染 markdown 提案。
  - `--json`：解析为 JSON 写到 stdout（含结构 + deltas）。
  - `--requirements-only` / `--deltas-only`：只输出 ADDED / MODIFIED / REMOVED / RENAMED 部分。
  - 无参数 + TTY：进入交互选择。
- **`openspec change list`**：扫 `openspec/changes/`，列出所有 pending change，支持 `--json`。
- **`openspec change validate <id>`**：解析 change 文件并用 Zod schema 校验，保证 delta 结构合法。

### 2. 交互与非交互回退
- TTY 下不传参数 → 显示交互列表 → 用户选择 → 继续。
- 非交互（`--no-interactive` / `OPEN_SPEC_INTERACTIVE=0` / stdin 非 TTY）下不传参数 → 打印"现有 hint + 可用 IDs" + `process.exitCode = 1`，**不**执行任何操作。
- 这样 CI/脚本里行为可预测，本地体验不被干扰。

### 3. Legacy 兼容
保留旧的 `openspec list` 行为，并在每次执行时打印 deprecation 提示 `Note: 'openspec list' is deprecated. Use 'openspec change list' instead.`。`--all` 等 flag 沿用旧语义。
- 该 deprecation 提示正是 verb-noun 迁移（见 openspec-conventions 的"Verb-Noun CLI Command Structure"）的过渡安排。

### 4. 与 `cli-show` 的关系
`cli-show` 是顶层 `show` 的统一入口，能根据名字自动判定 change 还是 spec；`cli-change` 是 noun-first 旧路径。两者**都**指向同一底层实现（spec 显式说"maintain all existing show options"）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec change {show,list,validate} [id]` |
| 输出 | Markdown / JSON / 仅 deltas（--deltas-only） |
| 交互 | TTY + 无参数 = 列表选择；非 TTY 走 hint+exit 1 |
| 校验 | Zod schema 解析 + 结构 + delta 合法 |
| 兼容 | `openspec list` 仍可用但提示 deprecation |
