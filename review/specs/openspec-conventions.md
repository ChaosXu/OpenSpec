# openspec-conventions

## 目的 (Purpose)

`openspec-conventions` 是 OpenSpec 自身的元规范 (meta-specification)，定义了系统能力的描述方式、变更提案与跟踪流程，以及规范随时间演化的方式。它是 OpenSpec 自身惯例的"事实来源 (source of truth)"。

该规范回答了几个核心问题：
- spec/change 的目录结构应该长什么样？
- 应该如何书写结构化的 Requirement + Scenario？
- 变更应该如何以 delta (ADDED / MODIFIED / REMOVED / RENAMED) 形式存储和归档？
- 命名、能力边界、变更生命周期等约束有哪些？

## 内容解析 (Content Analysis)

### 1. 规范格式约束
- **结构化格式 (Structured Format)**：每个 spec.md 必须使用 `### Requirement: [Name]` 加 `#### Scenario: [Description]` 的层次结构；场景步骤使用 `**WHEN** / **THEN** / **AND** / **GIVEN**` 加粗关键字。
- **行为优先 (Behavior-First)**：spec 描述"可观察的行为契约"——接口、错误处理、约束，不写实现细节（类名、库选型等应在 design.md / tasks.md 中）。
- **渐进式严谨 (Progressive Rigor)**：低风险本地变更用简洁 spec；跨团队、API 破坏、迁移、安全敏感等高风险变更才提高详细度。

### 2. 标识与匹配机制
- **Header-Based Requirement Identification**：需求标题就是唯一标识符，归一化方式为 `trim()`，再做大小写敏感比较。
- **重命名支持**：通过 `## RENAMED Requirements` 段配 `FROM: ... → TO: ...` 显式声明。
- **唯一性约束**：同一 spec 内不能出现重复 header，校验工具必须报错。

### 3. 变更存储约定
- **Delta 存储**：change 提案只存增量（ADDED / MODIFIED / REMOVED / RENAMED），不存完整 future state。
- **标准输出符号**：`+` (ADDED, 绿)、`~` (MODIFIED, 黄)、`-` (REMOVED, 红)、`→` (RENAMED, 青)。
- **归档流程顺序**：先 RENAMED → 再 REMOVED → 再 MODIFIED（按新名） → 最后 ADDED；冲突需手动解决。
- **Proposal 显式描述**：用 `From / To / Reason / Impact` 字段清晰记录每次变更（弥补 delta 不内嵌 diff 的不足）。

### 4. 项目结构
定义了 `openspec/` 目录的标准布局（specs / changes / archive），以及每个 change 必须包含的 `proposal.md` + `tasks.md` + 可选 `design.md` + `specs/<capability>/spec.md`。

### 5. 变更生命周期
Propose → Review → Approve → Implement → Deploy → Update → Archive，七步流程；每步都有清晰的语义。

### 6. CLI 词汇约束
- 顶层命令采用 **Verb-Noun** 结构（如 `openspec list`）；旧式 noun-first（`openspec spec` / `openspec change`）保留至少一个版本并提示弃用。
- 出现重名歧义时用 `--type spec|change` 显式声明。

### 7. Workspace 产品语言
最新的扩展部分（约 2026-05）：
- 文档统一用 "workspace = cross-repo planning home" 的产品语言，避免 "working set / code area / entry / alias / local overlay" 等内部术语。
- 称被影响的 repo/package/service 为 **affected areas**，不用 "target repo" / "repo slice"。
- 称实现切分为 **slices / phases**，仅在讨论交付顺序时使用。
- 规划与实现边界：workspace 是"共享规划家"，repo-local 仍是"代码与权威行为的家"。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 文件结构 | `openspec/specs/<cap>/spec.md` + `openspec/changes/<id>/{proposal,tasks,design,specs}.md` |
| Spec 语法 | `### Requirement:` + `#### Scenario:` + WHEN/THEN/AND |
| 变更格式 | Delta（ADDED/MODIFIED/REMOVED/RENAMED），无 diff 语法 |
| 命名 | 能力名用 kebab-case 单一职责；CLI 用 verb-noun |
| 工作流 | 探索→提案→审批→实施→部署→同步→归档 |
