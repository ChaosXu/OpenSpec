# schema-fork-command

## 目的 (Purpose)

定义 `openspec schema fork` 命令——把已存在的 schema（包内置 / 用户 / 项目本地）**复制**到项目本地的 `openspec/schemas/<name>/`，作为"先 fork 再改"的工作流基础。

## 内容解析 (Content Analysis)

### 1. 复制
- `openspec schema fork <source> [name]`：
  - 显式名 → 写到 `openspec/schemas/<name>/`，更新 `schema.yaml` 的 `name` 字段。
  - 缺名 → 默认 `<source>-custom`。
  - source 找不到 → 报错列出可用 schema，exit ≠0。

### 2. 防止误覆盖
- 目标已存在：
  - 交互模式 → 提示确认。
  - `--force` → 删除旧目录后覆盖。
  - 都没 → 报错 + 建议 `--force`，exit ≠0。

### 3. 完整复制
- 包括 `schema.yaml`、所有模板文件、嵌套子目录（如 `templates/specs/`）。
- 文件内容不变。

### 4. JSON 输出
- `--json`：
  - 成功：`{ forked: true, source, destination, sourcePath, sourceLocation }`（`sourceLocation` 是 `project` / `user` / `package`）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec schema fork <source> [name]` |
| 复制 | 整个 schema 目录 + 嵌套结构 |
| 覆盖 | `--force` 或交互确认；默认拒绝 |
| 输出 | `--json` 含 `forked / source / destination / sourcePath / sourceLocation` |
