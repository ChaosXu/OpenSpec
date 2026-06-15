# cli-view

## 目的 (Purpose)

`openspec view` 提供一个**项目状态仪表盘**——一次性把 specs、active changes、draft changes、completed changes、整体任务进度等关键指标渲染成统一、可视化的视图。目的是让开发者"打开项目第一眼"就能感知 OpenSpec 健康度。

## 内容解析 (Content Analysis)

### 1. 仪表盘布局
无 `openspec/` 目录 → 报错 `✗ No openspec directory found`。
有则按四段渲染：Summary / Active Changes / Completed Changes / Specifications（以及独立的 Draft Changes 段，见下文）。

### 2. Summary 段
- 总 spec 数 + 总 requirement 数
- draft change 数（无 `tasks.md` 或 0 task 的 change）
- 活动 change 数（进行中）
- 已完成 change 数
- 整体任务完成百分比

空项目全 0。

### 3. Active Changes 显示
按完成百分比**升序**排序（0% 优先），无 task 的视作 0%；同进度按 change id 字母升序，输出确定。

### 4. Completed Changes 显示
- **修正 bug**：原先 `total === 0` 会被误判为完成，现在要求 `tasks.total > 0` 且 `tasks.completed === tasks.total` 才算。
- 用 `✓` 标记放在独立段。

### 5. Draft Changes 段
针对 `tasks.md` 缺失或 0 task 的 change，独立放在 Draft 段，用 `○` 标记，按字母升序。

### 6. Specifications 段
按 requirement 数量**降序**展示，并显示计数标签；解析失败的 spec 计为 0 仍展示（不静默吞掉）。

### 7. 视觉格式
- 颜色：spec 用青、active 用黄、completed 用绿、辅助文字用 dim gray。
- 进度条：完成部分 `█`、剩余 `░`。
- 解析或文件系统失败时：跳过坏数据、继续渲染（优雅降级）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec view` |
| 分段 | Summary / Draft / Active / Completed / Specifications |
| 排序 | Active 按完成度升序；Draft / spec 按字母或 requirement 降序 |
| 视觉 | 色 + 进度条 + 符号（✓ ○） |
| 健壮性 | 跳过坏数据继续渲染；显式区分 "no tasks" vs "completed" |
