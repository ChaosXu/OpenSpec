# rules-injection

## 目的 (Purpose)

定义 `config.yaml` 的 `rules` 字段如何**注入到对应 artifact 指令**——按 artifact id 精准匹配、XML 风格标签包裹、**叠加**而非替换 schema 自带指令，并对未知 artifact id 发出警告。

## 内容解析 (Content Analysis)

### 1. 精准匹配
- 只有当 `rules[artifactId]` 存在时，才为该 artifact 注入 `<rules>` 段。
- 没匹配项 / 没 `rules` 字段 / 空数组 → **不**输出 `<rules>` 标签。

### 2. 标签格式
- `<rules>\n- rule1\n- rule2\n</rules>\n\n`（每条规则一个 bullet）。
- 顺序：`<context>` → `<rules>` → `<template>`。

### 3. 内容保真
- Markdown 格式（`**bold**`）、特殊字符（`<` `>` 引号）、多行字符串 → **原样保留**，不转义不解释。

### 4. 叠加而非替换
- schema 自带的 artifact 指令 **保留**。
- config 里的 rules **额外添加**——agent 同时看到"schema 模板 + 项目规则"。

### 5. 多 artifact 支持
- 不同 artifact 可有不同 rule 集。
- 有 rule 的 artifact 显示，无 rule 的不显示（**不**共享）。

### 6. 未知 artifact id 警告
- 在 instruction load 时（此时 schema 已知）校验 `rules` 里的 key。
- 未知 → 警告 `Unknown artifact ID in rules: '<id>'. Valid IDs for schema '<schema>': design, proposal, specs, tasks`。
- 多个未知 → 每条单独警告。
- 警告按 id 去重，每个 CLI 会话里每条只显示一次（缓存）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 匹配 | 按 artifact id 精准；没匹配就不输出 |
| 标签 | `<rules>` 包 bullet 列表 |
| 位置 | `<context>` 之后，`<template>` 之前 |
| 关系 | **追加**到 schema 指令，不替换 |
| 校验 | 未知 id 警告；会话内去重 |
