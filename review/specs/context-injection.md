# context-injection

## 目的 (Purpose)

定义项目 context（`config.yaml` 的 `context` 字段）如何**注入到所有 artifact 指令**中，用 XML 风格 `<context>` 标签包裹、原样保留。确保 AI 写 proposal / specs / design / tasks 时都能看到项目级的"上下文说明"。

## 内容解析 (Content Analysis)

### 1. 注入范围
- 所有 artifact 指令（proposal、specs、design、tasks）都注入。
- config 没 `context` 字段或为 undefined → **不**输出 `<context>` 标签。
- 多行 context 保留换行。

### 2. 标签格式
- 严格格式：`<context>\n{content}\n</context>\n\n`。
- 在 `<template>` 之前出现。

### 3. 内容保真
- 特殊字符（`<` `>` `&`、引号、URL、Markdown 格式如 `**bold**` / `[link](url)`）**原样保留**——不转义、不渲染、不解释。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 触发条件 | config.yaml 有 `context` 字段 |
| 标签 | `<context>\n...\n</context>\n\n` |
| 位置 | 在 `<template>` 之前 |
| 内容 | 原样保留，不转义不解释 |
| 范围 | 所有 artifact 指令 |
