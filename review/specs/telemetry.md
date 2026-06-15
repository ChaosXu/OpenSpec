# telemetry

## 目的 (Purpose)

定义 OpenSpec 的**匿名使用遥测**——通过 PostHog 收集事件以改进产品，明确"什么会发、什么不会发、用户怎么退出、何时禁用"。原则：最小化、透明、尊重隐私。

## 内容解析 (Content Analysis)

### 1. 事件设计
- **唯一事件**：`command_executed`，属性只有 `command`（如 `init` / `change:apply`）和 `version`。
- **绝不**包含：命令参数、文件路径、项目名、spec 内容、错误信息、IP 地址。
- 显式设 `$ip: null` 阻止 PostHog IP 跟踪。

### 2. 退出机制
- `OPENSPEC_TELEMETRY=0` → 全程不发。
- `DO_NOT_TRACK=1` → 全程不发。
- `CI=true` → 自动禁用（CI 优先于显式 `OPENSPEC_TELEMETRY=1`）。
- 三者优先级：环境变量 > 配置文件 > CI 自动禁用。

### 3. 首次运行提示
- 第一次执行命令时，若 telemetry 启用且 `noticeSeen: false`，打印一行：
  `Note: OpenSpec collects anonymous usage stats. Opt out: OPENSPEC_TELEMETRY=0`
- 提示在**任何事件发送前**出现。
- 后续执行不再提示。

### 4. 匿名身份
- 首次发事件时生成 UUID v4，存到 global config 的 `telemetry.anonymousId`。
- 跨 session 复用同一 ID。
- **用户先 opt-out**：永远不生成 anonymousId（懒生成）。

### 5. 发送策略
- `flushAt: 1` + `flushInterval: 0` → 即时发送，不批量。
- CLI 退出前 await `posthog.shutdown()`，无论成功 / 失败都执行。

### 6. 静默失败
- 网络错误 / PostHog 宕机 / `shutdown()` 超时 → **不抛错**，CLI 命令正常完成。
- 遥测永远不能阻塞用户工作流。

### 7. 与 feedback 的关系
- `openspec feedback` 不走 telemetry 通道（直接调 `gh`），且 telemetry 关闭时 feedback 仍能提交。
- CI 模式下 feedback 也照常（如果 `gh` 可用且已认证）。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 后端 | PostHog |
| 事件 | `command_executed`（只含 `command` + `version`） |
| 禁用 | `OPENSPEC_TELEMETRY=0` / `DO_NOT_TRACK=1` / `CI=true` |
| 身份 | UUID v4 懒生成，存 global config |
| 发送 | 即时（flushAt=1） |
| 失败 | 静默，永不阻塞 CLI |
| 提示 | 首次一行告知如何退出 |
