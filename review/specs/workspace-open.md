# workspace-open

## 目的 (Purpose)

定义 `openspec workspace open` 命令——把一个 workspace "打开"成"工作集"，通过选中的 opener（agent / VS Code 编辑器）启动，同时把 linked repo/folder 作为可见工作集呈现。是 workspace-foundation / workspace-links 的"启动器"。

## 内容解析 (Content Analysis)

### 1. Workspace Open 命令
- `openspec workspace open`：
  - 当前在 workspace 内 → 打开当前 workspace + 用 workspace 的 selected opener。
- `openspec workspace open <name>`：打开指定 workspace。
- `openspec workspace open --workspace <name>`：同上。
- **冲突选择器**：同时传 `<name>` + `--workspace` 名字不一致 → 报错指明两个冲突选择器。
- **不支持 flag**：`--prepare-only` / `--json` → 明确报错，引导到未来的 context/query 表面。
- **不支持 change-scoped open**：`--change <id>` → 报错，引导到未来 workspace change planning 工作流。

### 2. Workspace 解析
- **当前 workspace 优先**：在 workspace 目录内运行 + 无 name → 用当前 workspace。
- **唯一 known workspace 自动选**：在 workspace 外 + 只有一个 → 直接打开。
- **多个 → picker**：交互显示 workspace 名 + 位置。
- **多个 → 非交互报错**：列出已知名 + 建议传 name。
- **无 known workspace**：建议 `openspec workspace setup`。

### 3. Opener 解析
- **冲突 flag**：`--agent codex --editor` → 报错 + 不启动 + 不改 stored opener。
- **stored opener 优先**：默认用 `.openspec-workspace/view.yaml` 里的 `preferred_opener`。
- **临时覆盖**：
  - `--agent codex` → 本次用 codex，**不**改 stored。
  - `--editor` → 本次用 VS Code 编辑器，**不**改 stored。
- **没 stored opener + 交互**：prompt 选 opener（**只**列检测到可执行的）。
- **没 stored opener + 交互但无可执行**：报错不弹（避免给出"点了不能跑"的选项）。
- **没 stored opener + 非交互**：报错 + 提示传 `--agent <tool>` 或 `--editor`。

### 4. Opener 启动行为
- **VS Code 编辑器**：`code` 在 PATH → 打开 workspace 的 `<workspace-name>.code-workspace`。
- **GitHub Copilot in VS Code**：`code` 在 PATH → 同样打开 `.code-workspace`，作为 Copilot 体验。
- **Codex**：`codex` 在 PATH → 从 workspace 根启动 + 附上所有合法本机路径的 linked repo/folder（用 Codex 自己的目录附加机制）。
- **Claude**：`claude` 在 PATH → 同上，附 linked 路径。
- **缺可执行**：报错 + 指明缺哪个 + 维持 required opener 不变。
- **缺 `code`（VS Code / Copilot）**：报错 + 给 `.code-workspace` 路径以便用户手动打开。

### 5. Linked Working Set 可见性
- **合法本机路径的 link** → 全部纳入"打开的工作集"（前提是 opener 支持目录附加）。
- **支持**"还没有 workspace change 时就打开"（探索阶段）。
- **坏路径**：跳过 + 报告 `openspec workspace doctor` 是修复路径 + 继续打开（只要 opener 本身可用）。
- **缺 repo-local OpenSpec** → 仍把 link 纳入（实现就绪度是后续流程的事）。

### 6. Workspace Open Guidance
- **已有 `AGENTS.md` 指引** → 刷新 `.code-workspace` 反映当前 link 状态 → 用刷新的 workspace 文件启动。
- **必须传初始 prompt 的 opener** → 用最小 prompt 如 `Open this OpenSpec workspace.`；workspace 规则保留在 workspace 文件里。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec workspace open [name] [--workspace ...] [--agent ...\|--editor]` |
| 解析 | 当前 workspace > 唯一 known > picker > 报错 |
| Opener | stored 优先；flag 临时覆盖；冲突报错 |
| 启动 | VS Code / Copilot / Codex / Claude 各自的目录附加机制 |
| 健壮性 | 缺可执行报错；坏 link 跳过 + doctor 提示；缺 repo-local OpenSpec 仍纳入 |
| 不支持 | `--prepare-only` / `--json` / `--change`（明确报错并引导未来工作流） |
