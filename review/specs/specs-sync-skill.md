# specs-sync-skill

## 目的 (Purpose)

定义 `/opsx:sync` agent skill 的对外行为——把 change 中的 delta spec（ADDED/MODIFIED/REMOVED/RENAMED）同步到主 `openspec/specs/`，是 `cli-archive` 在 agent 端的对等物。目的是让 AI agent 在 archive 之外也能手动触发"把变更合并回主 spec"。

## 内容解析 (Content Analysis)

### 1. 同步流程
- 读 change 目录 `openspec/changes/<name>/specs/` 下的 delta。
- 读对应能力的主 spec `openspec/specs/`。
- 按 delta 操作头归并主 spec。

### 2. Delta 调和逻辑
- **ADDED**：主 spec 缺 → 新增；主 spec 已有 → 用 delta 版本**覆盖**（视为修复）。
- **MODIFIED**：按 normalized header 找到主 spec 对应 requirement → 替换为 delta 整段。
- **REMOVED**：按 header 移除。
- **RENAMED**：按 `FROM: / TO:` 改名；若内容也改，作为 MODIFIED 用新名再走一遍。
- **新能力 spec**：delta 描述了主 specs 没有的能力 → 创建 `openspec/specs/<capability>/spec.md`。

### 3. 幂等性
- 多次跑同一 change → 结果与单次一致；不产生重复 requirement。

### 4. 选择交互
无 change 名时，prompt 用户选择——只列**有 delta specs**的 change。

### 5. 输出
- 同步成功 → 按能力展示 added/modified/removed/renamed 数量。
- 已同步 → `Specs already in sync - no changes needed`。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | agent 执行 `/opsx:sync [change]` |
| 操作 | ADDED（含覆盖已存在）/ MODIFIED / REMOVED / RENAMED |
| 幂等 | 多次跑结果一致；不重复 |
| 输出 | 按能力分组 + 计数 + "already in sync" 提示 |
