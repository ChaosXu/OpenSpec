# cli-validate

## 目的 (Purpose)

定义顶层 `openspec validate` 命令及输出规范：校验 change 与 spec，给出**可操作的修复建议**（不是冷冰冰的错误码）以及结构化（JSON）输出，目标是把"validate 失败 → 不知道怎么改"的卡点变成"看到错误就知道下一步"。

## 内容解析 (Content Analysis)

### 1. 校验输出哲学
- **可操作的修复步骤 (Actionable Remediation)**：每个错误都附带
  - 期望结构 / header 样例；
  - 引用 `openspec/AGENTS.md` 里的 quick-reference；
  - 调试命令（如 `openspec change show {id} --json --deltas-only`）。
- **场景格式侦测**：识别"散落的 WHEN/THEN/AND 项目符号"并提示改用 `#### Scenario:` 头。
- **结构化位置**：每个 issue 都带 `file`（如 `openspec/changes/{id}/specs/{cap}/spec.md`）+ `path`（如 `deltas[0].requirements[0].scenarios`），方便跳转。
- **Next steps 收尾**：校验失败时 human-readable 模式附加 Next steps 区块，含 summary 行 + top-3 引导 + 重跑命令建议。

### 2. 顶层 validate 命令
- **交互（TTY + 无参）**：让用户选「all / changes / specs / specific item」→ 执行。
- **非交互（`--no-interactive` / `OPEN_SPEC_INTERACTIVE=0` / 非 TTY）**：打印 hint + 可用 flag 列表 + exit 1。
- **直接给名字 `<item>`**：唯一匹配自动检测；同时匹配 change+spec → 歧义（提示 `--type`）；都不匹配 → not-found + nearest-match。
- **`--type change|spec <item>`**：显式类型，跳过自动检测。

### 3. 批量与过滤
- `--all`：校验 `openspec/changes/`（排除 archive/）+ `openspec/specs/`，展示 passed/failed 摘要，任一失败 → exit 1。
- `--changes`：仅校验 change。
- `--specs`：仅校验 spec（要求存在 `openspec/specs/<id>/spec.md`）。

### 4. 选项与进度
- `--strict`：warning 视为 error，有 warning 即失败。
- `--json`：输出机器可读 JSON。
- **JSON 模式 schema**：
  - `items[]`: `{ id, type: "change"|"spec", valid, issues[], durationMs }`。
  - `summary.totals`: `{ items, passed, failed }`。
  - `summary.byType.change?` / `summary.byType.spec?`：分组统计。
  - `version`: schema 版本字符串（"1.0"）。
  - 任一 `items[].valid === false` → exit 1。
  - 单条 `Issue`: `{ level: "ERROR"|"WARNING"|"INFO", path, message }`。
- **进度显示**：批量校验时显示 spinner / 当前正在校验的项 + 通过/失败计数。
- **并发限制**：4–8 并发，保证进度反馈仍响应。

### 5. 跨平台
Markdown parser 兼容 LF / CRLF / CR 行尾；CRLF 保存的 change proposal 也能正确识别 `## Why` / `## What Changes` 等必要节。

### 6. 交互控制
- `--no-interactive` 显式关闭。
- `OPEN_SPEC_INTERACTIVE=0` 环境变量同样关闭。
- 提示仅在 stdin 是 TTY 且未禁用时显示。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec validate [<item>] [--all|--changes|--specs] [--type ...] [--strict] [--json]` |
| 错误信息 | 包含 file path + 结构化 path + 修复引导 |
| 摘要 | Next steps 区块（human 模式）；summary 对象（JSON 模式） |
| 性能 | 批量并发 4–8，进度条响应式 |
| 跨平台 | 兼容 LF / CRLF / CR |
