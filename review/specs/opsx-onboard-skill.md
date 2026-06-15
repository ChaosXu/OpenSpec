# opsx-onboard-skill

## 目的 (Purpose)

定义 `/opsx:onboard` agent skill——带领用户**第一次走完完整的 OpenSpec 流程**（探索 → 新建 → 起草制品 → 实施 → 归档），用真实代码库的工作做教学，目标是让新人 15 分钟内"上手即会"。

## 内容解析 (Content Analysis)

### 1. 引导流程
- 检查是否初始化；未初始化则提示先 `openspec init`。
- 开场：欢迎消息 + ~15 分钟预期 + 流程阶段列表。

### 2. 任务建议
- 扫描代码库找小改进点：TODO/FIXME、缺错误处理、未测试函数、过时依赖、`any` 类型、生产代码 `console.log`、缺输入校验。
- 看近期 git commit 找上下文。
- 呈现 3-4 条具体建议 + 范围估计（文件数/行数）+ "为什么是好的起点" + "自己给任务"的选项。
- 用户挑的任务太大 → 温和劝分小，给替代小任务。

### 3. 探索阶段演示
- 简要演示 `/opsx:explore`（不是完整探索会话），告诉用户 explore 是"先想后做"。

### 4. 制品创建引导
逐步带创建 change 目录 → proposal.md → specs → design.md → tasks.md，每步：
- 解释该制品的目的（change / proposal = WHY / specs = WHAT / design = HOW / tasks = checklist）。
- 实时起草，**先给用户看再保存**。
- 等用户确认再下一步。

### 5. 实施
- 宣布每条任务再做。
- 偶尔回顾 specs/design 是怎么影响决策的（不啰嗦）。
- 完成即勾选。

### 6. 归档讲解
- 解释 archive 把 change 移到 dated 目录、便于将来找决策。

### 7. 收尾
- 总结完成的阶段。
- 强调"这个节奏适用任何规模的变更"。
- 给命令速查表（`/opsx:explore` `/opsx:new` `/opsx:ff` `/opsx:continue` `/opsx:apply` `/opsx:verify` `/opsx:archive`）。
- 建议下一步（"试试 `/opsx:new` 或 `/opsx:ff`"）。

### 8. 优雅退出
- 用户想停 → 确认 + 告知 in-progress change 已保存 + 教 `continue <name>`。
- 用户只想看命令 → 给 cheat sheet + 鼓励 `try /opsx:new`。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `/opsx:onboard` |
| 流程 | 探索→新建→制品→实施→归档 |
| 教学 | 每步有 narration；起草先给用户看再保存 |
| 范围 | 自动找小任务 + scope guardrail |
| 退出 | 优雅支持中途停止 + 速查分支 |
