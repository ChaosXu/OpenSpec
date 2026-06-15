# workspace-foundation

## 目的 (Purpose)

定义 OpenSpec **workspace（协调工作空间）** 的产品语义和存储底座——workspace 身份、共享 vs 本机状态、托管存储位置、本地注册表、稳定链接名、repo 所有权边界。是后续 workspace-links / workspace-open / workspace-change-planning 的根基。

## 内容解析 (Content Analysis)

### 1. Recognizable Workspace Home
- workspace 是**跨 repo 规划的稳定家**。
- 必须在 workspace 根或其子目录运行 → 才进入 workspace 模式。
- `changes/` 子目录存在不等于 workspace（避免误判）；必须有 `.openspec-workspace/` 身份文件。

### 2. Stable Workspace Name
- 全 OpenSpec 共用同一 kebab-case 名：`.openspec-workspace/view.yaml` / 托管 workspace 目录名 / 本地注册表名。
- 拒绝：空、`.`、首尾连字符、连续连字符、大写、空格、下划线、`.`、路径分隔符。
- 文件夹创建失败要清晰报错。

### 3. Workspace vs Repo-Local
- workspace 用 `.openspec-workspace/`，repo-local OpenSpec 用 `openspec/`，两者并存时各司其职。
- linked repo 仍保留 `openspec/`，不被 workspace 入侵。
- 不需要在 workspace 根目录里再 init 一份 repo-local `openspec/`。

### 4. Safe Sharing
- 共享信息（workspace 身份 + 稳定 link 名）便携，**不含**绝对 checkout 路径。
- 本地路径存为机器本地状态（`view.yaml` 私有），换机器可重新映射。
- 支持原生 Windows 路径、WSL2/Linux 路径。
- `view.yaml` 是私有本地视图：stable link 名 + 本机路径值。

### 5. Standard Workspace Location
- 托管 workspace 存 `<global-data-dir>/workspaces`（沿用 XDG 平台目录）。
- **不**在当前 slice 引入 workspace 专属 env / 命令 / 配置来改这个位置。
- Windows（PowerShell）→ Windows global data 位置 + 原生路径。
- WSL2 → Linux/XDG 位置 + Linux 路径。
- 创建时**报告**位置给用户，不隐藏。

### 6. Local Workspace Registry
- 机器本地的 workspace 索引（仅 name + location）。
- **workspace 文件夹本身是 source of truth**，注册表只是索引。
- 后续命令可用注册表做 picker。
- stale entry（注册表里指向已不存在的目录）**仅报告**，不静默删，不自动修复，不暴露 `workspace forget` 之类的清理命令。

### 7. Stable Link Names
- workspace 状态和规划制品引用 repo/folder 时**只用 stable link name**，不用本机绝对路径。
- 同一 link 名跨机器可映射到不同本机路径。
- 拒绝：空、`.`/`..`、含路径分隔符。
- workspace 内 link 名必须唯一。
- 不强制 link 名遵循 workspace-name 的更严 kebab-case（允许 folder-style 名）。

### 8. Linked Repos & Folders
- linked repo/folder **不必**有 repo-local `openspec/`——workspace 规划可以先于 repo 采纳 OpenSpec。
- monorepo 内的多个 package/service/app/folder 可分别链接。
- 不同 repo + monorepo folder 用同一种规划模型。
- 链接**只**记录到 workspace 状态，**不**创建/复制/移动/初始化/编辑 linked repo/folder 里的文件。

### 9. Planning Before Implementation
- 创建/检测 workspace 是规划动作。
- repo 实施文件**不**动——必须显式走后续 workspace 工作流。
- apply / verify / archive 都需要显式后续工作流。

### 10. Repo Ownership Boundaries
- workspace 计划引用某个 repo 拥有的行为 → 那个 repo 仍是 canonical spec 与实现工作的"家"。
- workspace 让跨边界计划**可见**，但不夺取所有权。
- 探索阶段，workspace 可持有草案，**与** canonical repo-owned specs 区分。

### 11. Preferred Opener State
- 用户在 setup 时选过 opener → 存到 `.openspec-workspace/view.yaml` 的 `preferred_opener: { kind, id }`。
- 交互 setup：让用户选；非交互 `openspec workspace setup --no-interactive --opener codex` → `kind: agent` / `id: codex`。
- 非交互不传 `--opener` → opener 留空，让 `workspace open` 之后弹窗。
- 接受值：`codex` / `claude` / `github-copilot` / `editor`（`editor` 映射为 `kind: editor` / `id: vscode`）。
- 交互 setup 列 opener 时，**已检测到可执行**的排前；不可用也保留显示 + 标注"不可用"。

### 12. Maintained Workspace Open Surface
- `openspec workspace setup` 完成后创建/刷新 `AGENTS.md` 与 `<workspace-name>.code-workspace`。
- `workspace link` / `relink` 成功后也刷新。
- `<workspace-name>.code-workspace` 内容：
  - 所有合法本机路径的 linked repo/folder 在前，workspace 本地文件在后；
  - 关联 initiative context（如果有）；
  - workspace 根标为 `OpenSpec workspace`；
  - 缺/无效本机路径的 link **省略**。
- **清理**遗留的 ignore rule for the maintained `.code-workspace` 文件；其他用户写的不动。
- `AGENTS.md` 已有内容：只替换 OpenSpec workspace guidance marker 块；marker 缺失则**追加**。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 身份 | `.openspec-workspace/view.yaml` |
| 命名 | workspace 名 kebab-case；link 名可 folder-style（稳定） |
| 共享 vs 本地 | 共享不含绝对路径；本机路径在 view.yaml |
| 位置 | `<global-data-dir>/workspaces`（标准 XDG / Windows 数据目录） |
| 注册表 | 本地索引 + 报告 stale，不自动清理 |
| Repo 边界 | workspace 不动 linked repo 内部 |
| 实施 vs 规划 | workspace 是规划；实施走显式后续工作流 |
| Open surface | `AGENTS.md` + `.code-workspace`，按需刷新 |
