# cli-archive

## 目的 (Purpose)

`openspec archive` 把已完成的 change 从 `openspec/changes/` 移到 `openspec/changes/archive/YYYY-MM-DD-<name>/`，**同时**将 change 的 future-state spec（delta）合并回主 `openspec/specs/`，确保 `specs/` 始终反映"已部署的真实状态"。目的是在保持历史可追溯的同时，让主 specs 永远与代码同步。

## 内容解析 (Content Analysis)

### 1. 命令语法
```
openspec archive [change-name] [--yes|-y]
```
- 不传 change 名：进入交互选择列表（排除 archive/）。
- `--yes/-y`：跳过所有确认提示（CI 友好）。

### 2. 任务完成检查
- 扫 `tasks.md` 的 `- [ ]`，发现未完成项**列出**并提示用户确认（默认 No，保证安全）。
- 全部完成或无 `tasks.md` 则直接进入归档。

### 3. 归档流程（按顺序）
1. 创建 `archive/` 目录（若不存在）。
2. 生成 `YYYY-MM-DD-<change-name>` 目标名（用当前日期）。
3. 目标已存在 → **报错不覆盖**（保护历史）。
4. **将 change 中的 delta specs 应用到主 `openspec/specs/`**（调用 openspec-conventions 的归档流程：RENAMED → REMOVED → MODIFIED → ADDED）。
5. 把整个 change 目录 mv 到 archive 目标。

### 4. Spec 更新（核心安全网）
- **应用 delta 前先校验**：违反 openspec-conventions 的合并规则（如 duplicate header）→ 报错并 abort。
- **冲突处理**：出现 MODIFIED/REMOVED 的 header 在主 spec 里找不到、或 ADDED 的 header 已存在 → abort 并提示人工解决。
- **逐 spec 显示变更**：用 `+`/`~`/`-`/`→` 符号（与 conventions 对齐）展示 added/modified/removed/renamed 数量。

### 5. 确认行为
- 弹窗清晰列出 "NEW specs to be created" 与 "EXISTING specs to be updated"，并附 source 路径。
- 默认 No（必须显式 `y`/`yes`）；`--yes` 跳过。
- 拒绝确认时：原版本 abort 整个流程并 exit ≠0；后来改为**跳过 spec 更新但继续归档**（更符合"我反正要归档"的实际诉求）。

### 6. 跳过 Specs 的开关
`--skip-specs` 完全跳过 spec 发现与确认，直接归档（适用于用户已经手动合并过的情况）。

### 7. 强制归档（危险模式）
`--no-validate` 跳过验证（unsafe mode），并显示警告。

### 8. 错误处理
统一处理：缺 `openspec/changes/`、change 不存在、archive 目标已存在、文件系统权限问题。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec archive [name] [--yes] [--skip-specs] [--no-validate]` |
| 行为 | 校验任务→应用 delta→mv 到 `archive/YYYY-MM-DD-<name>/` |
| 安全 | 目标存在不覆盖；delta 应用前必校验；冲突时 abort |
| 输出 | 标准符号（`+ ~ - →`）逐 spec 展示 |
| 决定性 | 默认要求确认，`--yes` 跳过；spec 更新可被拒绝但归档继续 |
