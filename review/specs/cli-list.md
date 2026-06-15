# cli-list

## 目的 (Purpose)

`openspec list` 是开发者的"快速概览面板"，扫描 `openspec/changes/` 给出每个 change 的名称和任务完成情况；通过 `--specs` 切换到列出当前已部署的 spec 及其 requirement 数量。目的是"不开多个文件也能看项目进度"。

## 内容解析 (Content Analysis)

### 1. 两种扫描模式
- **默认 (--changes)**：扫 `openspec/changes/`，排除 `archive/`，解析每个 change 的 `tasks.md` 统计任务完成度。
- **`--specs`**：扫 `openspec/specs/`，读取每个能力目录的 `spec.md`，解析 Requirement 计数。

### 2. 任务计数规则
- `- [x]` 计为已完成
- `- [ ]` 计为未完成
- 总数 = 已完成 + 未完成
- 全部完成显示 `✓ Complete`

### 3. 输出格式
- **change 列表**：表格，列：change 名称 | 任务进度（如 `3/5 tasks`）
- **spec 列表**：表格，列：spec id | requirement 数（如 `requirements 12`）

### 4. 标志位
- `--specs`：列出 spec。
- `--changes`：显式列出 change（与默认行为相同）。
- 旧式 `--all` 等行为已废弃（参见 cli-change spec）。

### 5. 空状态
- 无活动 change：`No active changes found.`
- 无 spec：`No specs found.`

### 6. 错误处理
- change 目录没有 `tasks.md`：该行显示 `No tasks`。
- `openspec/changes/` 目录不存在：报错 `No OpenSpec changes directory found. Run 'openspec init' first.`，exit 1。

### 7. 排序
多 change 时按名称字母升序，输出稳定可预测。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec list`（默认 changes）/ `openspec list --specs` |
| 任务解析 | `- [x]` / `- [ ]` 计数 |
| 输出 | 表格化，状态清晰 |
| 错误 | 缺目录→exit 1，缺 tasks.md→显示 `No tasks` |
| 排序 | 字母升序 |
