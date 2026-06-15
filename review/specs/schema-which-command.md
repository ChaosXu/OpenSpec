# schema-which-command

## 目的 (Purpose)

定义 `openspec schema which` 命令——报告**指定 schema 解析到了哪里**（来源 + 路径 + 是否 shadow 了低优先级同名 schema），是 schema-resolution 的人工诊断工具。

## 内容解析 (Content Analysis)

### 1. 基本报告
- `openspec schema which <name>`：
  - 来源 `project` / `user` / `package` + 完整路径。
  - 找不到 → 报错 + 列出可用 schema，exit ≠0。

### 2. Shadowing 信息
- 当高优先级来源覆盖低优先级时，显示：
  - 当前激活的是 project 版本。
  - 它 shadow 了 package 版本（列出被 shadow 的路径）。
  - 多层 shadow（如 project shadow user 和 package）按优先级全部列出。
  - 无 shadow → 不显示该段。

### 3. JSON 输出
- 基本：`{ name, source, path }`。
- 含 shadow：`shadows[]`（每条 `{ source, path }`）。

### 4. List 模式
- `openspec schema which --all`：列出所有可用 schema + 来源 + shadow 关系。
- `--all --json`：JSON 数组。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec schema which <name> [--json]` / `--all` |
| 报告 | source + 路径 + shadow 链 |
| Shadow | 多层时按优先级全列 |
| 输出 | human / `--json` |
