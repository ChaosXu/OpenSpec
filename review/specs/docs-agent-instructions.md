# docs-agent-instructions

## 目的 (Purpose)

为 OpenSpec 自动生成 / 维护的 AI agent 指令文档（`openspec/AGENTS.md`）制定**写作标准**——让模板、示例、校验清单**清晰、可复制、即取即用**，而不仅仅是"漂亮的散文"。

## 内容解析 (Content Analysis)

### 1. Quick Reference 优先
`openspec/AGENTS.md` 必须以 quick-reference 段开头，把 `proposal.md` / `tasks.md` / spec deltas / scenario 格式的**可复制 heading** 摆在最前面，让 agent 第一眼拿到模板。每条模板还要链向对应工作流步骤的详细章节。

### 2. 内联模板 + 示例
写工作流引导时，**就地**给出 fenced markdown 模板（`## Why` / `## ADDED Requirements` / `#### Scenario:` 等），并配简短示例展示正确 header 与 scenario bullet 写法。

### 3. 校验前清单
在讨论 `openspec validate` 之前给一个**简短的预校验清单**，提醒：
- Requirement header 格式
- Scenario 用 `#### Scenario:` 而非散落 bullet
- 必要的 delta section
- scenario 前的描述性 Requirement 文本

### 4. 渐进式披露
- 入门段：只保留"脚手架→起草→校验→请求评审"四步。
- 高级段：移到后部分（如多能力变更、归档细节、工具深入），并用 anchor link 互相引用。

### 5. 行为优先的 spec 撰写指导
明确告诉 agent：
- spec 写"可外部验证的行为"——输入/输出/错误/约束。
- **不要**在 spec 里写库选型、类结构、函数签名。
- 那些细节去 `design.md` / `tasks.md`。

### 6. 渐进式严谨
- 例行变更：轻量、简洁。
- 高风险（API 破坏、迁移、跨团队、安全/隐私敏感）：使用更完整、更详尽的风格。
- 强调"在写 spec 之前先想清楚**最小可测试 / 可评审**的 spec 是什么"。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 文档位置 | 模板/示例与对应工作流段同地出现 |
| Quick Reference | 必须出现在文档开头 |
| 预校验清单 | 紧跟 validate 引导 |
| 渐进披露 | 入门 + 高级分段 + 锚点链接 |
| 行为 vs 实现 | 明确分流到 spec.md vs design/tasks.md |
| 严谨度 | 按变更风险分级 |
