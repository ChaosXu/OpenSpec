# cli-config

## 目的 (Purpose)

为 global-config 提供用户友好的 CLI 前端（`openspec config ...`），让用户**不必手改 JSON** 也能查看/修改全局配置。同时承载 `config profile` 这个 action-first 交互流程（修改交付方式 + 工作流）和 workspace 模式下的 apply 提示。

## 内容解析 (Content Analysis)

### 1. 子命令总览
- `path`：显示配置文件绝对路径。
- `list`：以 YAML-like 形式列出所有当前值；`--json` 输出纯 JSON。
- `get <key>`：用 camelCase + 点号（`featureFlags.someFlag`）；不存在 → exit 1 + 空输出；对象值 → 渲染为 JSON。
- `set <key> <value>`：自动类型推断（`true`/`false` → bool；数字 → number；其余 string）；`--string` 强制字符串；自动创建中间对象。
- `unset <key>`：移除该 key（恢复默认）；不存在 → 提示 + exit 0。
- `reset`：必须 `--all`；`--all` 触发前要求确认，`-y` 跳过。
- `edit`：在 `$EDITOR` / `$VISUAL` 中打开配置文件（不存在则用默认内容创建）；两者都未设 → 报错 + 提示设 `$EDITOR`。

### 2. 键名与 schema
- 使用 camelCase 匹配 JSON 实际字段名。
- 点号支持嵌套（`featureFlags.someFlag`）。
- **`set` 写入需通过 zod schema 校验**：未知 key 默认拒绝（exit 1），`--allow-unknown` 显式接受；非法值（如 `featureFlags.someFlag notABoolean`）拒绝。

### 3. `--scope` 标志保留
- 默认 `global`（即全局配置）。
- `--scope project` 显式报错 "Project-local config is not yet implemented"（为未来扩展留口）。

### 4. `config profile`（重点新增）
Action-first 交互：
- **第一屏**展示当前 profile summary（当前 delivery + workflow 数 + profile 标签）。
- **菜单**：
  - Change delivery + workflows
  - Change delivery only
  - Change workflows only
  - Keep current settings (exit)
- **Delivery 选项**标记 `[current]` 并预选。
- **No-op 路径**：选了"Keep current"或最终未改变有效值 → 打印 `No config changes.`，**不写**配置文件，**不弹** apply 提示。
- **No-op 但项目不同步**：仍打印 `No config changes.`，但加非阻塞警告"项目内文件未同步"，提示 `openspec update`。
- **有改动时** → 保存到全局 config → 在 OpenSpec 项目里再问 `Apply changes to this project now?`；用户确认 → 跑 `openspec update`。

### 5. Workspace 模式下的 `config profile`
- 配置仍保存到 global。
- **改 profile/delivery 后**：提示 `Apply changes to this workspace now?`；用户确认 → 跑 `openspec workspace update`，**不**跑 repo-local `openspec update`。
- **拒绝 apply**：告知用户"global 已改"，指引 `openspec workspace update`。
- **No-op 时不弹 apply 提示**，但如果 workspace-local skills 与 global profile 漂移，给非阻塞警告。
- **`config profile core` 快捷**：workspace 中改时只存 global、不弹 apply；repo-local 中保留旧行为（弹 apply 跑 `openspec update`）。
- **Workspace planning home 优先**：当当前位置在 workspace planning home 时，即使存在可被识别的 repo-local OpenSpec，也按 workspace 处理；反之，从 linked repo 里运行则保留 repo-local 行为。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec config {path,list,get,set,unset,reset,edit,profile}` |
| 类型 | set 自动类型推断；`--string` 强制；`--allow-unknown` 放行未知 key |
| Schema | zod 校验写入；非法值拒绝 |
| Profile | action-first 菜单；no-op 不写不 apply；workspace 模式分流 |
| 安全 | reset 必须 `--all` + 确认（`--all -y` 跳过） |
| 兼容 | `--scope project` 保留但未实现 |
