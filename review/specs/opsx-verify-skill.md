# opsx-verify-skill

## 目的 (Purpose)

定义 `/opsx:verify` agent skill——对 change 的**实施**做完整性、正确性、一致性三维评估，输出**有优先级（CRITICAL / WARNING / SUGGESTION）的报告**，并提供可操作的修复建议，目标是"归档前最后一道关卡"。

## 内容解析 (Content Analysis)

### 1. 调用
- `verify <change>`：直接校验指定 change。
- 不带名：prompt 用户选（**只**列有实施任务的 change）。
- 选中的 change 无 tasks.md 或 tasks 为空 → 提示 `No tasks to verify` + 建议 `/opsx:continue` 创建任务。

### 2. 完整性（Completeness）
- 读 `tasks.md` 统计 `- [x]` / `- [ ]`，列未完成项。
- 有 delta specs 时抽取 requirement，搜索代码看是否实现，列出实现 vs 缺失。
- 全部完成 → 标"pass"，输出 `Tasks: N/N complete`。
- 有未完成 → 标"CRITICAL"，附"完成剩余任务或标记为已完成"建议。

### 3. 正确性（Correctness）
- 对每个 requirement：
  - 在代码库搜实现 → 给出文件 + 行号 → 评估是否满足。
- 对每个 scenario：
  - 查代码是否处理了 scenario 条件；
  - 查是否有覆盖该 scenario 的测试。
- 实现匹配 spec → 标"covered"。
- 实现偏离 spec → 标"WARNING"，解释偏离，建议"改实现或改 spec"。
- 找不到实现 → 标"CRITICAL"，附"Implement requirement X" + 需要什么。

### 4. 一致性（Coherence）
- 有 `design.md` → 抽取关键决策 → 检查实现是否遵循 → 偏离标 WARNING。
- 无 `design.md` → 跳过该检查，注"No design.md to verify against"。
- 决策被遵循 → 标 confirmed 并引用代码证据。
- 决策被违反 → 标 WARNING，附"改实现或 design.md"。
- 另查新代码是否符合项目既有 pattern，偏离为 SUGGESTION。

### 5. 报告格式
- Summary 表格（Completeness / Correctness / Coherence 三维）。
- Issue 按优先级排序：CRITICAL → WARNING → SUGGESTION。
- 每条 issue 含**具体可操作**的修复建议（含文件:行号），避免"consider reviewing"这种空话。
- 全 pass → `All checks passed. Ready for archive.`
- 有 CRITICAL → `X critical issue(s) found. Fix before archiving.`（不暗示可以 archive）。
- 仅 warning → `No critical issues. Y warning(s) to consider. Ready for archive (with noted improvements).`

### 6. 灵活处理缺制品
- 只有 tasks → 只查 completeness。
- 有 tasks + specs，无 design → 查 completeness + correctness，跳过 design 一致性。
- 完整 change（proposal / design / specs / tasks）→ 全部三维 + 交叉引用一致性。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `/opsx:verify <change>` |
| 三维 | 完整性 + 正确性 + 一致性 |
| Issue 等级 | CRITICAL（必须修）/ WARNING（应修）/ SUGGESTION（可改） |
| 报告 | Summary 表 + 按优先级排序的 issue + 文件:行号引用 |
| 灵活 | 自动跳过不存在的制品，给出"已跳过"说明 |
