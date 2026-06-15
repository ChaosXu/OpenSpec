# command-generation

## 目的 (Purpose)

定义**工具无关**的 command 模板内容（`CommandContent`）与**工具专属**的格式化逻辑（`ToolCommandAdapter`）的契约，以及 `generateCommand` 与 `CommandAdapterRegistry` 组合机制——同一个 command body 在不同 AI 工具下生成对应的 frontmatter / 文件路径。

## 内容解析 (Content Analysis)

### 1. `CommandContent` 接口（tool-agnostic）
- `id`（如 `explore` / `apply`）
- `name`（如 `OpenSpec Explore`）
- `description`
- `category`（如 `OpenSpec`）
- `tags[]`
- `body`（共享指令文本）

### 2. `ToolCommandAdapter` 接口（per-tool）
- `toolId`：对应 `AIToolOption.value`
- `getFilePath(commandId)`：相对项目根的路径（Codex 这类全局工具可以是绝对）
- `formatFile(content)`：生成完整文件内容（含工具原生 frontmatter）

### 3. 工具示例
- **Claude Code**：YAML frontmatter `name` / `description` / `category` / `tags`；路径 `.claude/commands/opsx/<id>.md`。
- **Cursor**：`name` 用 `/opsx-<id>`，加 `id` / `category` / `description`；路径 `.cursor/commands/opsx-<id>.md`。
- **Windsurf**：同 Claude 风格；路径 `.windsurf/workflows/opsx-<id>.md`。

### 4. 生成器
- `generateCommand(content, adapter)` → `{ path, fileContent }`。
- 批量生成：遍历所有 command content，用同一 adapter。

### 5. `CommandAdapterRegistry`
- `get(toolId)` → 适配器或 undefined。
- `getAll()` → 全部已注册。
- 未注册 → 返回 undefined，调用方自行决定如何处理（skip / 报错）。

### 6. Body 共享
所有工具的 command **body 完全相同**，只有 frontmatter 与文件路径不同——保证"教学内容"在工具之间一致。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 内容层 | `CommandContent`（id/name/desc/category/tags/body） |
| 适配层 | `ToolCommandAdapter`（getFilePath + formatFile） |
| 工厂 | `CommandAdapterRegistry.get / getAll` |
| 共享 | body 跨工具一致；frontmatter 与路径由 adapter 决定 |
| 工具示例 | Claude / Cursor / Windsurf 各自 frontmatter 模板 |
