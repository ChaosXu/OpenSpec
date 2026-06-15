# workspace-links

## 目的 (Purpose)

定义 `openspec workspace` 命名空间下"**直接** workspace setup、列表、link、relink、doctor"等命令的对外行为，并提供结构化 JSON 输出与 workspace-local agent skill 安装/更新能力。是 workspace-foundation 的命令层落地。

## 内容解析 (Content Analysis)

### 1. Guided Setup（`openspec workspace setup`）
- 先问 workspace 名（kebab-case），再问 link 路径——**至少要 link 一个**才能完成。
- 非法 workspace 名 → 解释 + 让重输。
- **推断 link 名**：从路径 basename 推断，冲突才问。
- 冲突时显示已有同名 link 的路径，提示换一个。
- 允许 folder-style link 名（与 workspace-name 的更严 kebab-case 区分）。
- 可重复 prompt 持续加 link（**不**改 linked repo/folder 内部）。
- **验证绝对路径**：存的是已验证存在的绝对路径；相对输入相对 CWD 解析。
- **`--link <value>` 解析**：
  - 含 `=` 视为 `<name>=<path>`（path 内部允许 `=`）。
  - 不含 `=` 视为路径，link 名从 basename 推断。
- **非交互 setup**（`--no-interactive`）：必传 workspace 名 + 至少一个 link；重复 `--link` 接受。
- 重复 link 名 → 报错 + 列出冲突名 + 建议显式 `--link <name>=<path>`。
- 缺输入 → 报错 + 提示 flag。
- 完成时显示 workspace 位置 + planning path + linked repos/folders + 当前机器可解析情况。
- 完成后写入本地 workspace registry；workspace 文件夹仍为 source of truth。
- 重名 setup：解释已存在、**不**覆盖。

### 2. Discovery（`openspec workspace list` / `workspace ls`）
- 列出 known managed workspaces（含 name / location / linked repos/folders）。
- `ls` ≡ `list`。
- 无 workspace → 提示"未找到" + 引导创建。
- stale 注册条目（指向已不存在的目录）**报告但不删**；不静默清理、不自动修复。
- 当前 slice **不**暴露 `workspace forget` 这类清理命令。

### 3. Global Workspace Commands（任何目录都能跑）
- `--workspace <name>` 指定；未知名 → 报错。
- 没传但当前在 workspace 内 → 用当前 workspace。
- 当前在 workspace 内但未注册 → 仍用当前 workspace + 非阻塞警告 `workspace_not_in_local_registry`（教用户怎么补注册）。
- `workspace link` / `relink` 成功后**自动**注册当前 workspace。
- `workspace doctor` 不会自动注册。
- 多 workspace 候选：交互显示 picker（含 name + path）；非交互 → 报错建议 `--workspace <name>`；`--json` → 结构化 status error，**不**弹 picker。
- 无 known workspace + 不在 workspace 内 → `workspace link/relink/doctor` 等报错 `No known OpenSpec workspaces. Run 'openspec workspace setup' first.` 并提示 `--workspace <name>`。

### 4. Link（`openspec workspace link`）
- `<path>` → 推断 link 名（basename）+ 存绝对本地路径。
- `<name> <path>` → 用显式 link 名。
- 路径必须存在；否则清晰报错。
- 存的是"当前运行时的绝对路径"；相对输入相对 CWD；不跨 Windows/WSL/Unix 翻译。
- monorepo 内的 package/service/app/folder 可作 link，**不**要求自带 `openspec/`。
- linked 缺 repo-local OpenSpec → 不视为失败。
- **只记录**，不创建/复制/移动/初始化/编辑 linked repo 内容。
- 本地状态文件无法解析/校验 → 报错 `workspace_local_state_invalid`，**不**重写 shared/local 状态。
- 重复 link 名 → 报错 + 显示已有 link + 建议换名 / `workspace relink <name> <path>`（如果想换路径），**不**覆盖。

### 5. Relink（`openspec workspace relink`）
- `<name> <path>` → 保留稳定 link 名 + 更新本机路径。
- 新路径不存在 → 报错。
- 存的是"当前运行时的绝对路径"；相对输入相对 CWD。
- 本地状态损坏 → 报错 `workspace_local_state_invalid`，**不**重写。
- 未知 link 名 → 报错保留现有状态。
- **不**问 owner / handoff 元数据——本 slice 只关心 name + 本机路径。

### 6. Doctor（`openspec workspace doctor`）
- 默认检查**一个**选中的 workspace，**不**扫全部注册表。
- 选中规则：`--workspace <name>` / 推断当前 workspace。
- 健康状态显示：workspace 位置、planning path、每个 link 路径在本机是否可解析。
- 链接有 repo-local `openspec/specs/` → 报告 `repo_specs_path`；没有 → 报 `null`。
- 路径缺失 → 标识 link 名 + 建议 `workspace relink`。
- shared 与 local 状态漂移 → 解释哪些 link 名受影响 + 区分 shared vs local-only。
- 本地状态无法解析/校验 → 报 `workspace_local_state_invalid`，**不**当空 path map 处理，也**不**自动重写注册表/本地状态。
- **不**自动修复；只报告所有发现的问题。
- 人类输出走可读格式（不用裸 JSON、不用僵化 YAML dump）。

### 7. JSON 输出
- `--json` 走机器可读；不含多余人类文字；区分 primary objects 与 structured `status` 数组。
- `workspace setup --json` 必须搭配 `--no-interactive`，否则清晰失败。
- `--json` 禁用 prompt；歧义时输出结构化 status。
- 每个 status entry 至少含 `code` / `severity` / `message`；可附 `target` / `fix`。
- 对象可带 `status[]`（对象级 warning/error）；顶层响应也带 `status[]`（命令级）；健康时为空数组。
- 支持 JSON 输出的命令：`workspace setup --no-interactive` / `workspace list` / `workspace link` / `workspace relink` / `workspace doctor`。

### 8. Setup 安装 agent skills
- 交互 setup 走到 skill 安装时 prompt："workspace 里要为哪些 agent 装 OpenSpec skills"，用 **agent-skill** 措辞而非 "AI tools"。
- 用户选过 preferred opener 且支持 skill 生成 → 预选匹配 agent；可改选。
- 完成后在 workspace 根下为每个选中 agent 生成/刷新 OpenSpec skill 文件，并报告。
- 存储选中的 agent 到 workspace-local machine state。
- 若 global config 解析出 workflow profile → 装那个 profile 选中的 workflow（**不**用 `--tools` 选 workflow）。
- 装的是**workspace-local skills**，**不**装 slash command / global command（这部分本 slice 不做）。
- 即使 global config delivery 是 `commands` 或 `both`，仍只装 skills；提示"workspace command 生成不在本 slice"。
- 装 skills 不动 linked repo/folder。
- 非交互 `--tools all|none|<ids>` 走与 repo init 相同的合法 tool ID 集合校验。
- 非交互无 `--tools` → 不装 skills，提示 `openspec workspace update --tools <ids>`。
- JSON 输出含 generated/refreshed/skipped/failed 列表。

### 9. Workspace Update（`openspec workspace update`）
- 当前目录在 workspace 内 → 更新当前 workspace。
- `openspec workspace update <name>` / `--workspace <name>` → 更新指定 workspace。
- 完成后：
  - 刷新选中 agent 的 skills。
  - 新选中的 agent → 新建 skills。
  - 不再选中的 agent → 删 OpenSpec-managed workflow skill 目录（**仅**认带 `generatedBy` marker 的）。
  - 缺 marker 的一律不删。
  - 更新 workspace-local 选中的 agent 列表。
- profile-driven workflow 同步：profile 改 → workspace 同步到新 profile 选中的 workflow（仅删带 marker 的）。
- delivery = commands / both 仍只更新 skills。
- `--tools <ids>` / `--tools none` → 替换 workspace-local 选中的 agent 列表。
- 非交互 `--tools all|none|<ids>` → 跳过 prompt。
- 非交互无 `--tools` 但有 stored selection → 用 active global profile 刷新 stored。
- 非交互无 `--tools` 且无 stored selection → 完成（不装），提示用 `--tools`。
- 报告 drift：workspace-local last applied workflow IDs 与 active profile 不一致时报告，建议 `workspace update`。
- clean sync → 不报 drift。
- 更新结果报告（人类 + JSON 都有）：refreshed / added / removed / skipped / failed。
- `workspace --help` 列 `workspace update` 并描述"刷新 workspace-local agent skills"。
- `workspace update --help` 文档化 workspace 选择选项 + `--tools all|none|<ids>` + "global profile 选 workflow / `--tools` 选 agent"。
- shell completion 包含 `workspace update` 与相关 flag（`--workspace` / `--tools` / `--json` / `--no-interactive`）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `workspace {setup, list, ls, link, relink, doctor, update} [name/path] [--workspace ...] [--json] [--no-interactive] [--tools ...]` |
| 选择 | `--workspace` / 当前 workspace / picker；非 TTY 报错 |
| 路径 | 存绝对本机路径；相对 CWD 解析；不跨运行时翻译 |
| Stale | 报告但**不**自动清理；无 forget 命令 |
| 本地状态 | 损坏时 `workspace_local_state_invalid`，**不**自动修复 |
| Skills | 装 workspace-local；按 profile 选 workflow；`--tools` 选 agent；按 marker 识别可删 |
| JSON | 必带 `--no-interactive` 用于 setup；primary + status 分开 |
