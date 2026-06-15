# opsx-archive-skill

## 目的 (Purpose)

定义 `/opsx:archive` agent skill 的对外行为——在实验性工作流里归档已完成的 change，含制品完成度检查、任务完成度检查、spec 同步提示、归档执行和结果展示。CLI 版是 `openspec archive`，skill 版侧重让 AI 在引导下完成归档前后的所有确认。

## 内容解析 (Content Analysis)

### 1. 归档流程
- 检查 artifact 状态：所有 artifact 都 `done` → 跳过警告；否则列出未完成项 + 确认。
- 检查任务：从 `tasks.md` 读 `- [x]` / `- [ ]`；有未完成 → 警告 + 确认。
- 检查 delta specs：若 change 里有 `specs/` → 问用户是否先跑 `/opsx:sync`，无论选择如何都继续 archive。
- 执行 archive：建 `archive/`、生成 `YYYY-MM-DD-<name>/`、mv 整个 change 目录、保留 `.openspec.yaml`。
- 目标已存在 → 报错 + 建议改名或换日期。

### 2. 选择交互
无 change 名时 prompt 用户，只列**活动** change（排除 archive/）。

### 3. 输出
按场景分类：
- 同步过 + 归档成功 → 总结同步结果 + archive 位置 + 使用的 schema。
- 未同步 + 归档成功 → 提示 specs 未同步 + archive 位置 + schema。
- 含未完成项但仍归档 → 列出未完成内容 + 提示"是否有意为之"。

### 4. 与 cli-archive 的关系
底层共用 archive 流程；skill 在前面插入制品/任务/delta 的人工确认步骤，CLI 版是更"硬"的版本（任务未完成时默认拒绝并 abort）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | agent 执行 `/opsx:archive [change]` |
| 前置检查 | 制品完成度 + 任务完成度 + delta specs 同步 |
| 行为 | 三类确认都可继续；按用户意图 |
| 输出 | 总结：specs 同步 / 归档位置 / schema / 警告 |
| 关系 | 与 `openspec archive` CLI 共享底层；skill 偏向引导式 |
