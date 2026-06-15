# cli-spec

## 目的 (Purpose)

定义 `openspec spec` 命名空间子命令（`show` / `list` / `validate`）的对外行为——展示、列出、校验 source-of-truth 规范。与 `cli-change` 对称存在。

## 内容解析 (Content Analysis)

### 1. 三大子命令
- **`openspec spec show <id>`**：解析 `spec.md` → 层次化提取标题与内容 → 默认渲染 Markdown，可选 `--json`。
  - **过滤选项**：`--requirements`（只显示 Requirement 名称 + SHALL 声明）、`--no-scenarios`（隐藏 Scenario 子节）、`-r` / `--requirement <name>`（定位单条）。
- **`openspec spec list`**：扫 `openspec/specs/`，列出能力名，支持 `--json`。
- **`openspec spec validate <id>`**：用 Zod schema 校验 spec 结构（Requirements、Scenarios、WHEN/THEN 关键字等），报告结构问题。

### 2. JSON Schema 定义
通过 Zod schema 在运行时表达 spec 的结构（requirement / scenario / when-then-and 关键字），保证：
- 解析产物通过 schema 验证；
- 缺字段时报错清晰；
- 字段语义与 markdown 结构一一对应。

### 3. 交互与非交互回退
- TTY + 无参 `spec show` → 交互列表。
- 非 TTY / `--no-interactive` / `OPEN_SPEC_INTERACTIVE=0` → 打印原有 "missing spec-id" 错误，**非零退出码**。
- `spec validate` 同样适用：TTY 显示选择器，非 TTY 报错退出。

### 4. 与 `cli-show` / `cli-change` 关系
noun-first 旧路径；与顶层 `openspec show` 共用底层实现，保持所有 show 选项（`--json` / `--requirements` / `--no-scenarios` / `-r`）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec spec {show,list,validate} [id]` |
| 输出 | Markdown / JSON（`--json`）/ 过滤（`--requirements` / `--no-scenarios`） |
| 校验 | Zod schema 表达 spec 结构 |
| 交互 | TTY + 无参 = 列表；非 TTY = 错误退出 |
| 兼容 | 与顶层 `openspec show` 共享实现 |
