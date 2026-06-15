# cli-update

## 目的 (Purpose)

`openspec update` 把 OpenSpec 在项目里的核心文件 (`openspec/AGENTS.md`、根级 `AGENTS.md`/`CLAUDE.md` stub、已存在的 AI 工具 slash command 文件) 刷新到最新模板。目标是新版本发布后，用户一行命令拿到最新的 AI 指令，而不影响其自定义内容。

## 内容解析 (Content Analysis)

### 1. 前置条件
- 项目根目录必须已有 `openspec/`（由 init 创建），否则提示先跑 `openspec init` 并 exit 1。
- 整个命令**不需要网络**，纯本地模板替换。

### 2. 核心文件更新
- **完全替换** `openspec/AGENTS.md` 为最新模板。
- 若根级 stub (`AGENTS.md` / `CLAUDE.md`) 存在，刷新其托管块内容，确保继续指向 `@/openspec/AGENTS.md`；若不存在则可以创建。
- **AI 工具配置文件**遵循"刷新而非创建"原则：仅修改 OpenSpec 托管标记之间的内容，保留用户自写内容；不主动创建 slash command、CLAUDE.md 等文件。

### 3. 工具级更新规则（每种工具一个 Scenario）
- **Antigravity / Cline / Continue / Crush / Cursor / Kilo Code / Windsurf**：刷新 OpenSpec 托管部分，保留工具自有 frontmatter；不补缺失文件。
- **Claude Code / Codex / GitHub Copilot / iFlow CLI**：使用 shared templates 刷新。
- **CodeBuddy Code**：YAML frontmatter 含 `description` 与 `argument-hint`（如 `[change-id]`）；保留用户定制。
- **Factory Droid**：YAML 模板保留 `$ARGUMENTS` 占位符让 droid 接收用户输入。
- **Gemini CLI**：仅替换 `<!-- OPENSPEC:START -->` ... `<!-- OPENSPEC:END -->` 之间的 `prompt = """..."""` 块，保留 TOML 顶层 `description` / `prompt` 结构。
- **OpenCode**：archive 模板的 frontmatter 中包含 `$ARGUMENTS` 占位（用 `<ChangeId>\n  $ARGUMENTS\n</ChangeId>` 结构），以便接收 change ID。
- **Codex**：刷新全局 prompt 目录（不在项目根下）的对应文件，保留 marker 块外的用户内容；缺失则不创建。

### 4. Archive 命令参数支持
- `/openspec:archive <change-id>`：模板让 AI 用 `openspec list` 校验 change ID 是否合法；不合法则 fail-fast。
- 不带参数：保持原行为（从上下文或 `openspec list` 自动识别）。
- OpenCode 模板的 archive 文件 frontmatter 必须带 `$ARGUMENTS`。

### 5. Workspace 模式分流
当 update 从 workspace 根目录或 workspace planning home 子目录运行时：
- **不能**当作 repo-local 项目处理，**不能**生成 repo-local 项目文件。
- 明确提示用户改跑 `openspec workspace update`。
- 从 repo-local 项目（即使位于 workspace 父目录之下）运行则保留原行为。

### 6. 错误处理
- 写文件失败：让错误自然冒泡，包含文件路径。
- AI 工具配置文件缺失：跳过，不创建。
- 自定义目录名（如 `openspec2/`）**不支持**，统一使用 `openspec`。

### 7. 成功准则
- 一条命令完成更新。
- 无版本检查（不查远端），结果确定性。
- 显示 ASCII 安全的成功消息。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec update`（也可指定 `<path>`） |
| 行为 | 全量替换 AGENTS.md；刷新 marker 块；不创建缺失工具文件 |
| 工具 | 14+ 工具的 slash command 各自独立场景；Kimi 跳过 commands |
| Workspace | 拦截，提示用 `openspec workspace update` |
| 网络 | 不需要 |
