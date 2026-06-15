# schema-resolution

## 目的 (Purpose)

定义 schema 的**解析与优先级**逻辑：project-local > user > package built-in；处理回退、列表、去重、错误提示；并为新 change 选择 schema 提供统一优先级链（CLI flag > change metadata > planning home default > project config > hardcoded default）。

## 内容解析 (Content Analysis)

### 1. 解析优先级
- 调用 `getSchemaDir(name, projectRoot)` 时：
  1. `./openspec/schemas/<name>/`
  2. `~/.local/share/openspec/schemas/<name>/`（XDG_DATA_HOME）
  3. package built-in
- 不传 `projectRoot`（向后兼容）→ 只查 user + package。

### 2. 辅助函数
- `getProjectSchemasDir(projectRoot)` → `<projectRoot>/openspec/schemas`。

### 3. 列表
- `listSchemas(projectRoot)`：合并三个来源，project 覆盖 user 时不重复。
- `listSchemasWithInfo(projectRoot)`：每个 schema 带 `source: 'project' | 'user' | 'package'`。
- `openspec schemas` 输出每个 schema + 来源标签。

### 4. 为新 change 选 schema 的优先级链
1. CLI flag `--schema`
2. change 元数据（`.openspec.yaml` 的 `schema` 字段）
3. planning home default（workspace planning → `workspace-planning`）
4. project config (`config.yaml` 的 `schema` 字段)
5. hardcoded default `spec-driven`

### 5. project-local schema 名字
- config 的 `schema` 字段可引用 `openspec/schemas/<name>/` 里的本地 schema。
- 引用不存在的名字 → 错误含 fuzzy match 建议 + 全部可用 schema 列表（区分 built-in / project-local）+ 修复引导（`Edit openspec/config.yaml and change 'schema: X' to a valid schema name`）。

### 6. 向后兼容
- 早于 config 特性存在的 change：仍按 change metadata 或 hardcoded default 解析。
- 后加 config 不影响老 change 各自绑定的 schema。

### 7. Workspace planning schema
- `workspace-planning` 是 package 内置 schema（可被高优先级覆盖）。
- 列出时包含并标"package"或"project"等。
- 在 workspace planning home 创建 change 且无显式 `--schema` → 用 `workspace-planning` 作为 planning-home default，**在 project/global config 之前**。
- 显式 `--schema <name>` 仍按正常解析校验。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 解析优先级 | project-local > user > package |
| 列表 | 合并三方 + 来源标签 + 去重 |
| 选 schema 链 | CLI > change metadata > planning home > config > hardcoded |
| 错误 | 含 fuzzy 建议 + 可用列表 + 修复指令 |
| Workspace | `workspace-planning` 作为 planning-home default |
