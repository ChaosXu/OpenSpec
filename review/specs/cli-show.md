# cli-show

## 目的 (Purpose)

定义顶层 `openspec show` 统一入口的对外行为：自动判断输入是 change 还是 spec、处理歧义、支持 JSON / 各类 type-specific flag。目的是在 verb-noun 重构后给用户提供一个"先试 show，不行再加类型"的友好兜底。

## 内容解析 (Content Analysis)

### 1. 顶层 show 命令
- **无参数 + TTY** → 提示选 type（change / spec）→ 列出该项 → 展示。
- **无参数 + 非 TTY / `--no-interactive` / `OPEN_SPEC_INTERACTIVE=0`** → 打印 hint（`<item>` 的两种调用方式）+ exit 1。
- **直接给名字 `<item>`**：
  - 唯一匹配 change 或 spec → 显示该项。
  - 同时匹配 → 歧义错误，建议 `--type change|spec` 或 `openspec change/spec show`。
  - 都未匹配 → not-found 错误 + nearest-match 建议。
- **`--type change <item>` / `--type spec <item>`**：跳过自动检测，按显式类型处理。

### 2. 输出格式
- **Markdown（默认）** vs **JSON (`--json`)**，JSON 包含解析后的元数据与结构。
- 类型专属 flag 透传：
  - Change-only：`--deltas-only`（旧 `--requirements-only` 视为 alias 并弃用）。
  - Spec-only：`--requirements` / `--no-scenarios` / `-r` / `--requirement`。
  - 不相关 flag：忽略并警告。

### 3. 交互控制
- `--no-interactive` 禁用提示。
- `OPEN_SPEC_INTERACTIVE=0` 环境变量同样禁用。
- 仅在 stdin 是 TTY 且未禁用时才显示交互提示。

### 4. 与 `cli-change` / `cli-spec` 的关系
顶层 `show` 是聚合入口；`openspec change show` / `openspec spec show` 是 noun-first 老路径；底层共享同一实现。Flag scoping 保证每个 type 的专属 flag 正确路由。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec show [<item>] [--type change|spec] [--json] [--deltas-only|--requirements ...]` |
| 行为 | 自动类型检测 + 歧义提示 + nearest-match 建议 |
| 交互 | TTY + 无参 = 选 type；非 TTY = 打印 hint+exit 1 |
| 兼容 | 透传 `change show` / `spec show` 的全部 flag |
| 类型检测 | 唯一匹配自动；歧义显式；未知报错 |
