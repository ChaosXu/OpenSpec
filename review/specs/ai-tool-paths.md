# ai-tool-paths

## 目的 (Purpose)

定义 AI 工具的**路径元数据**（`AIToolOption.skillsDir`），让 OpenSpec 在生成 skills 与 commands 时知道写到哪个项目子目录。

## 内容解析 (Content Analysis)

### 1. `AIToolOption.skillsDir` 字段
- 可选字段，指向项目本地**基础目录**（如 `.claude`）。
- 生成的 skills 写在 `<projectRoot>/<skillsDir>/skills/`，`/skills` 后缀按 Agent Skills 规范追加。

### 2. 工具覆盖
- **Claude Code**：`skillsDir: '.claude'`
- **Cursor**：`skillsDir: '.cursor'`
- **Windsurf**：`skillsDir: '.windsurf'`
- **Kimi CLI**：`skillsDir: '.kimi'`
- **无 skillsDir** 的工具：尝试生成 skills 时报错"不支持"。

### 3. 跨平台路径
- 始终用 `path.join()`，**禁止**硬编码 `/`。
- Windows / Unix 行为一致。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 字段 | `AIToolOption.skillsDir`（可选） |
| 路径 | `<projectRoot>/<skillsDir>/skills/...` |
| 跨平台 | `path.join()` 必用，不硬编码 `/` |
| 工具覆盖 | Claude / Cursor / Windsurf / Kimi |
