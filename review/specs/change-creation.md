# change-creation

## 目的 (Purpose)

为 OpenSpec 提供**程序化**创建与校验 change 目录的工具函数（`createChange` 等），并支持 repo-local 与 workspace 两种 planning home 的创建路径。

## 内容解析 (Content Analysis)

### 1. 基础创建
- `createChange(projectRoot, 'add-auth')` → 创建 `openspec/changes/add-auth/`。
- 重复创建 → 抛错。
- 父目录不存在 → 递归创建。
- 非法名字 → 抛校验错。

### 2. Change 名校验
- 必须 kebab-case。
- **接受**：`add-user-auth` / `add-feature-2` / `refactor`。
- **拒绝**：
  - 大写字母（`Add-Auth`）
  - 空格（`add auth`）
  - 下划线（`add_auth`）
  - 特殊字符（`add-auth!`）
  - 首尾连字符（`-add-auth` / `add-auth-`）
  - 连续连字符（`add--auth`）

### 3. Workspace-aware 创建（关键新增）
- **从 workspace 根目录创建** → change 写到 workspace planning path；不写到 linked repo 的 `openspec/changes/`；未传 `--schema` 时使用 `workspace-planning`。
- **从 workspace 子目录创建** → 解析当前 workspace 为 planning home，行为同上。
- **从 linked repo 内部创建** → 保留 repo-local 行为（即使这个 repo 是 workspace 链接），**不**自动升级为 workspace 范围 change。
- **非 workspace 中创建 repo-local change** → 保留原 `openspec/changes/` 路径。
- **非法 affected area**：传入的 area 名字如果不在 workspace links 里 → 拒绝并列出有效名字。
- **暂未确定 affected area 时**：仍允许创建 workspace change，area 留待后续识别。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 函数 | `createChange(projectRoot, name, options?)` |
| 命名 | 严格 kebab-case（含数字后缀） |
| 行为 | 重复报错；自动建父目录 |
| Workspace | 自动选择 planning home + `workspace-planning` schema |
| Repo 内部 | 保留 repo-local 行为，不被链接状态劫持 |
