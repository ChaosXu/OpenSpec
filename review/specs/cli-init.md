# cli-init

## 目的 (Purpose)

`openspec init` 命令在任何项目里创建完整的 OpenSpec 目录结构，并支持为多种 AI 编码助手（Claude / Cursor / Windsurf / Kimi 等）批量生成 skills 与 slash commands，目标是让用户通过一条命令完成 OpenSpec 落地。

## 内容解析 (Content Analysis)

### 1. 进度反馈
- 全程使用 ora spinners 提示「Creating OpenSpec structure…」「Configuring AI tools…」等关键节点，校验阶段静默进行。
- 失败时输出明确文字（成功用 ✔，错误用 ✗）。

### 2. 目录结构创建
执行后生成：
```
openspec/
├── config.yaml
├── specs/
└── changes/
    └── archive/
```

### 3. AI 工具配置（核心功能）
- **交互模式**：启动时显示 OpenSpec Logo 动画欢迎屏，给出可搜索的多选列表（已配置项标记 "configured ✓" 并预选，列表置顶）。
- **非交互模式**：`--tools all` / `--tools claude,cursor` / `--tools none` 三种格式；保留值 `all`/`none` 不能与具体 ID 混用，违反则 exit code 1。
- 选中工具后，生成 `.<tool>/skills/`（9 个 SKILL.md）与 `.<tool>/commands/opsx/`（9 个 slash command）。
- **extend 模式**：当 `openspec/` 已存在时，init 不会重建基础结构，直接进入「add or refresh」流程；用户不选任何原生支持工具时仍 exit 0。
- **Kimi CLI 特殊处理**：仅生成 skills（`skillsDir: .kimi`），不生成 slash command（无 command adapter），输出 `Commands skipped for: kimi (no adapter)`。
- 行为总结 + 分类输出（Created / Refreshed / Skipped）+ 计数 + 「Next steps」提示（`/opsx:new` `/opsx:continue` `/opsx:apply`）。
- 提示「restart IDE for slash commands to take effect」。

### 4. 退出码
- `0` 成功
- `1` 通用错误（包含 OpenSpec 已存在时未选任何工具）
- `2` 权限不足（保留）
- `3` 用户取消（保留）

### 5. Skill 与 Command 模板
固定 9 个 skill（openspec-explore / new-change / continue-change / apply-change / ff-change / verify-change / sync-specs / archive-change / bulk-archive-change），每个 SKILL.md 包含 YAML frontmatter 和操作指引。

Slash command 与 skill 一一对应，但通过 ToolCommandAdapter 适配为工具原生格式（路径、前缀、frontmatter 字段不同）。

### 6. 配置文件
- 仅当 `config.yaml` 不存在时创建；存在则保留并标记 "(exists)"。
- 写入默认 schema 设置。

### 7. 向后兼容
保留 `openspec experimental` 作为隐藏命令（不显示在 help 中），等价于 `openspec init`。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec init` + 可选 `--tools <all\|none\|ids>` |
| 产出 | `openspec/{config.yaml, specs/, changes/{,archive/}}` + 每个工具的 skills/commands |
| 交互 | 动画欢迎屏 + 可搜索多选 + (configured) 标记 |
| 安全 | extend 模式不覆盖 config.yaml；缺失工具文件则跳过 |
| 兼容 | 隐藏 `openspec experimental` 入口 |
