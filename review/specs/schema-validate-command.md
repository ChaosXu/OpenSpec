# schema-validate-command

## 目的 (Purpose)

定义 `openspec schema validate` 命令——校验 schema 的 YAML 语法、Zod 结构、模板存在性、依赖图合法性，给出可机器读的 JSON 报告。

## 内容解析 (Content Analysis)

### 1. 命令行为
- `openspec schema validate [name]`：
  - 显式名 → 校验指定 schema。
  - 不带名 → 校验 `openspec/schemas/` 全部；任一失败 exit ≠0。
  - 找不到 → 报错 exit ≠0。

### 2. 校验项
- **YAML 语法**：失败时给行号。
- **Zod 校验**：识别缺字段（如缺 `name`），并定位。
- **模板存在**：artifact 引用的 template 文件必须真实存在；缺失 → `Template file '<x>' not found for artifact '<y>'`。
- **依赖图 DAG**：循环依赖 → 报错并列出涉及 artifact；未知依赖 → `Artifact '<x>' requires unknown artifact '<y>'`。

### 3. JSON 输出
- 成功：`{ valid: true, name, path }`。
- 失败：`{ valid: false, issues[] }`，每条 `{ level, path, message }`（与 `openspec validate` 输出一致）。

### 4. 详细模式
- `--verbose`：显示每个 check 的 pass / fail（YAML 解析、Zod 校验、模板存在、依赖图）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec schema validate [name]` |
| 校验 | YAML 语法 + Zod 结构 + 模板存在 + 依赖 DAG |
| 输出 | human（带行号）/ `--json`（与 `openspec validate` 对齐） |
| 模式 | 详细：`--verbose` |
