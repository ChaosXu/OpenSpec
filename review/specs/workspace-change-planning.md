# workspace-change-planning

## 目的 (Purpose)

定义如何在 workspace 里**创建、跟踪、引导 workspace 级 change**——其规划制品（proposal / specs / design / tasks）协调多个 linked repo 或 folder，**在实现所有权确定之前**让团队共享规划。

## 内容解析 (Content Analysis)

### 1. Workspace Change Planning Home
- 在 workspace 内创建 change → change 落在 workspace planning path，workspace 是该 change 的 planning home。
- 缺显式 `--schema` → 用 `workspace-planning` schema。
- 制品结构（无需额外 area manifest）：
  - workspace 级 proposal
  - workspace-scoped specs
  - cross-area design
  - 协调 tasks
- 制品都在 workspace change root 下。
- 产品目标在 workspace change 层**只表达一次**，**不**要求先在每个 linked repo 写 repo-local proposal。
- **不**在 linked repo/folder 内创建 repo-local change 目录；**不**编辑 linked repo 实现文件。

### 2. Workspace Affected Areas
- 所有权/实现边界用 **affected areas** 表示。
- 用注册的 workspace link 名作为 area 名 → 用 workspace links 校验，**非法 area 名清晰报错**。
- **探索阶段**：affected area 还没敲定前也允许存在 shared plan；未解决的 area 问题在 planning 制品 + status 输出里**保持可见**。
- **按 area 整理 requirements**：支持 `specs/<area-or-repo>/<capability>/spec.md` 嵌套路径。
  - **不**要求在 `specs/` 之外再建 area 文件夹。
  - area-or-repo 路径段**保留**为 workspace planning 上下文，**不**扁平化为 repo-local capability。
- **区分 area vs delivery slice**：area 是"所有权边界"；slice/phase 是"交付节奏"——**不**为小跨 area change 强制定义 delivery slice。

### 3. Workspace Planning Source of Truth
- 实施开始前，**workspace change 计划是 single source of truth**。
- 探索期：agent 用 workspace 级 planning 制品作共享规划源；linked repo/folder 视为"可用上下文"而非"已承诺的实现目标"。
- 实施期：repo-local 实现需要**显式**的 workspace 实现工作流 + 选定的 affected area，**并**在实现编辑开始前暴露该 area 的 allowed edit root。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| Planning home | workspace change root；用 `workspace-planning` schema |
| 制品 | proposal / specs / cross-area design / 协调 tasks |
| Affected areas | 来自 workspace link 名；可探索期间未敲定 |
| Spec 路径 | `specs/<area-or-repo>/<capability>/spec.md`（保留嵌套） |
| 边界 | 探索期 workspace plan 是 SoT；实施期需显式工作流 + area + allowed edit root |
