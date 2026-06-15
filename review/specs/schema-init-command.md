# schema-init-command

## 目的 (Purpose)

定义 `openspec schema init` 命令——在 `openspec/schemas/<name>/` 下**创建项目本地**的 schema 骨架（`schema.yaml` + 默认模板），支持交互 / 非交互两种模式，并可顺便设成项目默认 schema。

## 内容解析 (Content Analysis)

### 1. 创建
- `openspec schema init <name>`：
  - 合法名（kebab-case）→ 建 `openspec/schemas/<name>/` + `schema.yaml`（含 name / version / description / artifacts） + 引用的模板文件。
  - 非法名（空格等）→ 报错 + 建议 kebab-case，exit ≠0。
  - 已存在 → 报错建议 `--force` 或 `schema fork`，exit ≠0。

### 2. 交互模式
- TTY 下 prompt 描述、artifact 多选（含 proposal / specs / design / tasks 等常见选项，每个带简介）。

### 3. 非交互模式
- `--description "..."` + `--artifacts proposal,tasks` → 不 prompt。

### 4. 设为项目默认
- 交互 → 确认后写 `openspec/config.yaml` 的 `defaultSchema: <name>`。
- `--default` → 直接设。
- `--no-default` → 跳过。

### 5. JSON 输出
- 成功：`{ created: true, path, schema }`，不显示 prompt / spinner。
- 失败：`{ error }`，exit ≠0。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec schema init <name>` |
| 命名 | kebab-case |
| 模式 | 交互（TTY） / 非交互（flags） |
| 默认 | 可顺手写 `config.yaml` 的 `defaultSchema` |
| 输出 | `--json` 支持 |
