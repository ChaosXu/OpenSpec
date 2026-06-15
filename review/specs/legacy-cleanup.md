# legacy-cleanup

## 目的 (Purpose)

定义在 `openspec init` / `openspec update` 流程中如何**检测、清理** OpenSpec 旧版本（init 前的实现）留下的工件（CLAUDE.md 中的 OPENSPEC 段、老式 slash command 目录、`openspec/AGENTS.md` 等），同时**保护**用户的非 OpenSpec 内容。

## 内容解析 (Content Analysis)

### 1. 旧工件检测
- **工具配置文件**：`CLAUDE.md` / `.cursorrules` / `.windsurfrules` / `.clinerules` / `.kilocode_rules` / `.github/copilot-instructions.md` / `.amazonq/instructions.md` / `CODEBUDDY.md` / `IFLOW.md` 等（旧 ToolRegistry 里的所有）。
- **老式 slash command 目录**：`.claude/commands/openspec/` / `.cursor/commands/openspec/`（旧版用 `openspec-*.md`）/ `.windsurf/workflows/openspec-*.md` 等。
- **OpenSpec 结构文件**：`openspec/AGENTS.md`、根 `AGENTS.md`（带 OPENSPEC marker）、`openspec/project.md`（只显示迁移提示，不删）。

### 2. 确认流程
- 检测到遗留 → 列出内容 → 提示 `Legacy files detected. Upgrade and clean up? [Y/n]`，默认 Yes。
- 确认 Y / 回车 → 清理。
- 确认 N → abort init。
- 非交互（`--no-interactive` / CI）→ exit 1 + 列出检测到的内容 + 提示用交互或 `--force`。

### 3. 配置文件的安全清理
- 只删 `<!-- OPENSPEC:START -->` 到 `<!-- OPENSPEC:END -->` 之间的块，**保留**用户其他内容。
- 整文件只有 OPENSPEC 块 → 删块，保留文件（即使文件全空）——配置文件的"壳"是用户项目的。
- 清理后去除连续空行。

### 4. 目录删除
- 老式 slash command 目录（如 `.claude/commands/openspec/`）→ 整个删，**不**删父目录。
- `openspec/AGENTS.md` → 删文件，**不**删 `openspec/` 目录。

### 5. project.md 迁移提示（特殊）
- `openspec/project.md` **不**自动删——它可能有用户项目文档。
- 在输出里给迁移提示：`Manual migration needed: → openspec/project.md still exists. Move useful content to config.yaml's "context:" field, then delete.`
- 提示鼓励用 `/opsx:explore` 让 AI 协助迁移。

### 6. 清理报告
- 列出每个清理动作（✓ Removed OpenSpec markers from CLAUDE.md / ✓ Removed .claude/commands/openspec/ (replaced by /opsx:*) / ✓ Removed openspec/AGENTS.md (no longer needed)）。
- 单独列出"需要手动迁移"段。
- 无遗留 → 跳过清理段，直接进 skill 设置。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 检测 | ToolRegistry + 老式 slash command 目录 + `openspec/AGENTS.md` |
| 确认 | 交互 Y/n（默认 Yes）；非交互或 N → abort |
| 块级清理 | 只删 marker 块，保留用户内容；整文件只剩 marker 也保留空文件 |
| 目录清理 | 整目录删；不删父级 |
| 特殊 | `project.md` 提示迁移不自动删 |
