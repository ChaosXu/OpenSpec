# global-config

## 目的 (Purpose)

定义 OpenSpec 如何**解析、读取、写入用户级全局配置**——`~/.config/openspec/config.json`（macOS/Linux）或 `%APPDATA%\openspec\config.json`（Windows）。该 spec 是用户偏好、feature flag、跨项目设置的存储底座（XDG Base Directory 规范），并保证向前/向后兼容（schema 演进不会破坏旧文件）。

## 内容解析 (Content Analysis)

### 1. 存储位置与格式
- 默认路径：`~/.config/openspec/config.json`（macOS/Linux）。
- 格式：可读可手改的 JSON。
- 至少包含 `telemetry` 段（`anonymousId` UUID + `noticeSeen` bool）与 `featureFlags` 段。

### 2. 目录路径解析（XDG + 平台回退）
- **`$XDG_CONFIG_HOME` 已设** → 用 `${XDG_CONFIG_HOME}/openspec`。
- **Unix/macOS 未设 XDG** → `~/.config/openspec`。
- **Windows** → `%APPDATA%\openspec`。

### 3. 加载（getGlobalConfig）
- 文件存在 + 合法 → 解析返回。
- 文件不存在 → **不创建**文件，返回默认配置（空 `featureFlags`）。
- 存在但非法 JSON → 返回默认配置 + stderr 警告。
- **不抛错**（配置问题不该阻塞 CLI）。

### 4. 保存（saveGlobalConfig）
- 自动创建父目录。
- 覆盖式写入（trust latest state）。
- 写原子性：覆盖既有文件。

### 5. 默认配置
`{ featureFlags: {} }`（最小可用，未来字段通过 schema merge 注入）。

### 6. Schema 演进（关键）
- 加载时**与默认值做深合并**：新字段即便旧 config 文件没有也能用。
- **未知字段保留**：用户手写的扩展字段不被吞掉。
- 字段值优先级：旧文件已存在的字段胜过默认值。

### 7. 与 telemetry 协同
`anonymousId` 第一次发 telemetry 事件时生成并持久化；`noticeSeen` 控制 first-run 提示是否再显示。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 路径 | `~/.config/openspec` (XDG) / `%APPDATA%\openspec` (Windows) |
| 格式 | JSON |
| 加载 | 缺/坏文件 → 默认配置 + 警告（不抛错） |
| 保存 | 自动建目录 + 覆盖式 |
| Schema 演进 | 深合并 → 缺新字段补默认；未知字段保留 |
| 默认值 | 至少 `featureFlags: {}` |
