# cli-feedback

## 目的 (Purpose)

定义 `openspec feedback` 命令——通过 `gh` CLI 在 OpenSpec 仓库创建 GitHub Issue；当 `gh` 不可用时优雅降级为"输出可手动提交的内容"，并提供 `/feedback` agent skill 做对话上下文增强。

## 内容解析 (Content Analysis)

### 1. 命令行为
- `openspec feedback "title" [--body "..."]`：
  - 调 `gh issue create` 提交到 OpenSpec 仓库。
  - 打 `feedback` 标签。
  - 显示创建后的 issue URL。
- **安全**：用 `execFileSync` 传**参数数组**，不通过 shell——`"`、`` ` ``、`$()` 等都当字面量。
- issue body 包含元数据：CLI 版本、平台（`os.platform()`）、时间戳、`---\nSubmitted via OpenSpec CLI`。
- **不**包含：用户文件路径、项目名、env、IP。

### 2. gh CLI 依赖
- Unix/macOS 用 `which gh` 检测。
- Windows 用 `where gh` 检测。
- **缺 gh / 未认证**：回退——打印分隔符包裹的内容（`--- FORMATTED FEEDBACK ---` / `--- END FEEDBACK ---`）、预填 issue URL（`https://github.com/Fission-AI/OpenSpec/issues/new?title=...&body=...&labels=feedback`），exit 0。
- 未认证：额外提示 `gh auth login`。
- 错误：透传 `gh` 的 stderr，exit 码等于 `gh` 的退出码。
- 网络错误：提示检查网络，exit ≠0。

### 3. 与 telemetry 解耦
- `OPENSPEC_TELEMETRY=0` 不影响 feedback 提交。
- `CI=true` 下 feedback 照常（gh 可用且已认证时）。

### 4. `/feedback` agent skill
- agent 收集对话上下文 → 起草反馈 → **匿名化**（路径→`<path>`、key→`<redacted>`、公司→`<company>`、人名→`<user>`、URL→`<url>` 除非公开）→ **必须**展示给用户并等显式确认 → 通过 `openspec feedback` 提交。

### 5. Shell 补全
- `fee<TAB>` → `feedback`。
- `--<TAB>` 提示 `--body`。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 入口 | `openspec feedback "<title>" [--body "..."]` |
| 提交 | `gh issue create`（参数数组，无 shell 注入） |
| 回退 | 缺/未认证 gh → 输出可手动提交内容 + 预填 URL，exit 0 |
| 元数据 | 版本、平台、时间戳；不含敏感信息 |
| 匿名化 | `/feedback` skill 必须脱敏后让用户确认 |
