# instruction-loader

## 目的 (Purpose)

为 schema 驱动的工作流提供**指令模板的加载、校验、富化（enrichment）**——把模板与 change 上下文（依赖状态、可解锁的下一个制品等）拼成 agent 真正可用的指令串。下游服务于 `openspec instructions <artifact>`、`openspec status` 等命令。

## 内容解析 (Content Analysis)

### 1. 模板加载
- `loadTemplate(schemaName, templatePath)` → 从 `schemas/<schemaName>/templates/<templatePath>` 读取模板。
- 文件不存在 → 抛错（含路径）。

### 2. Change 上下文加载
- `loadChangeContext(projectRoot, changeName)` → 返回：
  - `graph`：artifact graph
  - `completedSet`：已完成的 artifact id 集合
  - `schemaName`：所用 schema
  - `changeInfo`：change 自身信息
- 支持 `loadChangeContext(projectRoot, changeName, schemaName)` 显式指定 schema。
- **change 目录不存在** → 返回空 completed set（视为"刚创建"），不抛错。

### 3. 模板富化
对每个 artifact 输出包含：
- **元数据**：change 名、artifact id、schema 名、output 路径
- **依赖状态**：每个依赖显示 `done` / `missing`
- **解锁项**：完成本制品后会解锁哪些 artifact
- **根制品标记**：无依赖的制品明确标注"root"

### 4. 状态格式化
`formatStatus(...)` 输出可读视图：
- 全部完成 → 全 `done`
- 混合状态 → `done` / `ready` / `blocked` 三态
- 阻塞制品显示缺哪些依赖
- 每个 artifact 显示 output 路径模式

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `loadTemplate` / `loadChangeContext` |
| 富化项 | 元数据 + 依赖状态 + 解锁项 + 根标记 |
| 状态 | done / ready / blocked 三态 + 缺依赖明细 |
| 失败 | 缺模板抛错；缺 change 目录不抛错（空集） |
