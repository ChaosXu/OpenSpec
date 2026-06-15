# artifact-graph

## 目的 (Purpose)

定义 **artifact graph（制品图）模型**——schema 驱动的 change 工作流的核心数据结构。负责：加载 schema、计算构建顺序、检测依赖闭环、判断制品完成度、识别可创建/被阻塞的制品；并支持 project-local / user / package built-in 的 schema 解析与 `workspace-planning` 内置 schema。

## 内容解析 (Content Analysis)

### 1. Schema 加载
- 源：`schema.yaml`（在 schema 目录下）。
- 失败用例：
  - 缺必要字段 → 抛错。
  - 循环依赖 → 列出循环里的 artifact id。
  - `requires` 引用不存在的 id → 报错指明非法引用。
  - 重复 id → 报错指明重复项。
  - 目录不存在 → 报错列出可用 schema。

### 2. 构建顺序（topological sort）
- 线性链 A→B→C → `[A, B, C]`。
- 菱形 A→B, A→C, B→D, C→D → A 在前，B/C 在 A 之后，D 最后。
- 独立制品：稳定顺序（id 字母升序，保证可重现）。

### 3. 完成度状态检测
按 `generates` 字段扫描：
- 文件存在（如 `proposal.md`）→ 完成。
- 文件不存在 → 未完成。
- glob 模式（`specs/*.md`）：目录存在且有匹配文件 → 完成。
- change 目录不存在 → 全部未完成（空状态）。

### 4. Ready 查询（getNextArtifacts）
- 没有依赖的根制品（无任何依赖被满足）→ 一开始就 ready。
- 依赖全部完成的制品 → ready。
- 还有依赖未完成的 → 不返回。

### 5. 完成判定
- `isComplete()`：所有制品都在 completed set → true。
- `getBlocked()`：返回 `{ [id]: missingDeps[] }`，单依赖、多依赖、全部依赖都列。

### 6. Schema 目录结构
- 自包含：`schema.yaml` + 可选 `templates/`。
- 覆盖优先级：用户目录 (`${XDG_DATA_HOME}/openspec/schemas/<name>/`) > package built-in。
- 列出 schemas：合并用户 + package，project-local 也参与（在 schema-resolution spec 中定义）。

### 7. Workspace Planning Schema（关键新增）
- 内置 `workspace-planning` schema。
- 制品包含：共享 proposal、workspace-scoped specs、跨 area design、协调 tasks——**不**需要额外 area manifest。
- specs 制品的 `generates` 模式为 `specs/**/*.md`，支持 `specs/<area-or-repo>/<capability>/spec.md` 这种嵌套路径作为默认约定。
- 模板与指令引导 agent 写 workspace 级别内容，**禁止**让 agent 在该阶段创建 repo-local 实现制品。
- workspace-scoped 嵌套 spec 路径在 status / instructions 中保留具体路径，**不**扁平化为 repo-local。
- apply 准备：必须完成协调 tasks 后才能进入实现；apply 引导要求 agent 先选 affected area 再做实现编辑。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 模型 | 有向无环图（artifact 节点 + 依赖边） |
| 加载 | YAML schema；非法字段/循环/未知依赖/重复 id 全部报错 |
| 顺序 | 拓扑排序；同层稳定 |
| 状态 | 文件存在性 / glob 匹配 |
| 查询 | ready / blocked / complete |
| 覆盖 | 用户 > package；project-local 优先（在 schema-resolution 中定义） |
| Workspace | 内置 `workspace-planning` schema；嵌套 spec 路径保留 |
