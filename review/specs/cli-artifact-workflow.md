# cli-artifact-workflow

## 目的 (Purpose)

定义 artifact 驱动工作流的 CLI 表面：`status` / `instructions` / `templates` / `new change` / workspace setup 相关命令，以及 `apply` 阶段指令、setup 命令的 `--tool` 标志处理。整体把"制品图（artifact graph）"翻译成"agent 友好"的 CLI 体验，并扩展到 workspace 模式。

## 内容解析 (Content Analysis)

### 1. Status 命令
- 显示每个 artifact 的状态：
  - `[x]` 已完成
  - `[ ]` 就绪（ready）
  - `[-]` 被阻塞（附缺哪些依赖）
- 展示 `2/4 artifacts complete` 类摘要。
- `--json` 输出 `{ changeName, schemaName, isComplete, artifacts, applyRequires }`，并包含 `applyRequires` 列表。
- **修复 bug**：原先要求 `proposal.md` 存在才能识别 change（`getActiveChangeIds` 限制）；现在 scaffolded（空）change 也能 status——根制品显示 ready，依赖制品显示 blocked。
- 缺 `--change` 参数：报错 + 列出所有可用 change 目录（包含 scaffolded）。
- 未知 change id：报错列出所有目录。

### 2. Next Artifact Discovery
- 废弃单独的 "next command"，统一用 `openspec status --change <id>` 的 `[ ]` 标记识别下一步。

### 3. Instructions 命令
- `openspec instructions <artifact> --change <id>` 输出元数据 + 模板 + 依赖状态 + 解锁项。
- `--json` 输出匹配 `ArtifactInstructions` 接口。
- 未知 artifact → 报错列出当前 schema 的合法 id。
- 阻塞制品 → 给出"依赖未满足"警告。
- scaffolded change 也能 instructions（不需要任何已存在制品）。

### 4. Templates 命令
- `openspec templates`：列每个 artifact 的解析后模板路径（默认 schema）。
- `--schema <name>`：用指定 schema。
- `--json` 输出 `{ artifactId: path }`。
- 标注每条是"用户覆盖"还是"package built-in"。

### 5. New Change 命令
- `openspec new change <name>` 创建 `openspec/changes/<name>/`。
- 非法名字 → 校验错误 + 引导。
- 已存在 → 报错。
- `--description "..."` 写入 README.md。

### 6. Workspace Setup 命令（关键新增）
- 提供 `openspec workspace setup` / list / link / relink / doctor 等命令，**不**依赖已有 workspace change。
- 短命令：`openspec workspace ls` ≡ `openspec workspace list`。
- setup 与 "agent launch" / "workspace open" 解耦——setup 不要求选择 agent。
- **不**暴露 `openspec workspace create` 作为公开创建入口（统一用 `workspace setup`）。

### 7. Schema 选择
- 默认 `spec-driven`。
- `--schema <name>`：用指定 schema（必须存在，否则报错列可用）。
- 与 `change-creation` / `schema-resolution` 协作。

### 8. 输出格式
- 颜色：完成用绿、ready 用黄、blocked 用红。
- `--no-color` 或 `NO_COLOR` 环境变量 → 纯文本符号。
- 加载时显示 spinner。

### 9. 实验性隔离
- 所有命令集中在 `src/commands/artifact-workflow.ts`，方便日后移除。
- `--help` 标注 experimental。

### 10. Schema Apply Block
- schema 可定义 `apply` 块：`requires`（apply 前必有的 artifact id）/ `tracks`（进度跟踪文件，可为 null）/ `instruction`（apply 引导文本）。
- 无 `apply` 块 → 要求所有 artifact 完成后才可用 apply，默认 instruction `"All artifacts complete. Proceed with implementation."`

### 11. Apply Instructions
- `openspec instructions apply --change <id>` 输出：
  - `contextFiles`：artifact id → 现有具体文件路径数组
  - schema 特定 instruction
  - `tracks`：进度文件路径或 null
- 缺必要制品 → 提示 apply 被阻塞 + 列出要先创建哪些。
- `--json` 输出 `{ contextFiles, instruction, tracks, applyRequires }`。

### 12. Setup 的 `--tool` 标志
- `openspec artifact-experimental-setup --tool <id>`：在指定 AI 工具目录下生成 skills。
- 无 `--tool` → 报错列出合法 id。
- 工具无 `skillsDir` → 报错"不支持"。
- 工具有 `skillsDir` 但无 command adapter → 生成 skills、跳过 commands + 信息提示。
- 成功输出含 "Setting up for <Tool>..." + 列出所有生成/跳过的文件路径。

### 13. 状态 JSON 的规划上下文
针对 workspace 模式：
- 报告 planning home（repo-local 还是 workspace-scoped）。
- 报告 artifact 的具体路径（含 workspace 嵌套 `specs/<area>/<cap>/spec.md`，**不**扁平化）。
- 报告 affected areas 状态（可未解析）。
- 提供 next-step 引导（自然语言动作）。

### 14. Action Context
- **规划态**：JSON 标识 agent 可读 / 写的规划制品；声明 linked repos 是"探索上下文"而非"提交目标"。
- **实现态**：包含 selected affected area 的 allowed edit root；避免越权编辑其他 area。
- **Repo-local 态**：保留现有 artifact 状态行为；规划 home 标为 repo-local。

### 15. Instructions 使用解析后的路径
- Workspace artifact 指令：指向 workspace change root 下的路径，**不**让 agent 写 linked repo 除非显式实现上下文。
- Repo-local artifact 指令：保留原 repo-local 路径。

### 16. 工作流 skills 用 CLI 输出作 single source of truth
- Skills 在做 artifact 工作前先 `openspec status --change <id> --json` 拿规划上下文与路径。
- 在写 artifact 前先 `openspec instructions <artifact> --change <id> --json` 拿目标路径。
- **不**写死 `openspec/changes/<id>/` 这种假设。
- 暂未支持的 workspace 工作流：在 skill 里明确告诉 agent"暂不支持"，**不**让 agent 回退到 repo-local 路径。

### 17. Workspace schema 指令
针对 `workspace-planning` schema：
- Status 列出 proposal / specs / design / tasks 四个标准制品。
- Specs 指令引导把 area 特定需求组织在 workspace 范围内 `specs/` 路径下，**不**要求 area 全部敲定，**不**在 workspace 规划阶段让 agent 创建 repo-local spec 文件。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 命令 | `status` / `instructions` / `templates` / `new change` / `apply` 指令 |
| Schema 选择 | 默认 spec-driven；`--schema` 切换 |
| Workspace | setup/list/ls/link/relink/doctor；统一 `workspace setup` 入口 |
| Apply | `apply.requires` 控制可用性；`tracks` 跟踪进度；instruction 引导 |
| 状态 JSON | 含 planning home、具体路径、affected areas、action context |
| 健壮性 | 颜色 / spinner / experimental 标注 / scaffolded 兼容 |
