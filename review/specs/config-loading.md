# config-loading

## 目的 (Purpose)

定义 `openspec/config.yaml`（项目级配置）的**发现、解析、校验、暴露**流程，并在出错时**安全降级**——配置坏了不能让整个 CLI 不可用。

## 内容解析 (Content Analysis)

### 1. 加载位置
- 默认 `openspec/config.yaml`。
- `.yml` 扩展名可作为别名。
- 两者同时存在时优先 `.yaml`。

### 2. 容错解析（field-by-field）
**不**走"任一字段失败整文件拒绝"——逐字段独立校验：
- `schema`（字符串，可选）：
  - 缺 → 不警告。
  - 空串 → 警告 + 不包含。
  - 非字符串（如 123）→ 警告 + 不包含。
- `context`（字符串）：
  - 同样支持；类型错就警告 + 不包含。
- `rules`（`{ artifactId: string[] }`）：
  - 子键为数组 → 包含该规则集。
  - 子键非数组 → 仅警告该子键，**其他**子键仍生效。
  - 数组里有非字符串元素 → 过滤掉非法项 + 警告。
- 部分字段合法 + 部分非法 → 返回合法字段，警告非法。

### 3. 大小限制
- `context` 超过 50KB → 警告 + 不包含（避免注入巨型 spec 指令）。
- 50KB 临界值包含。

### 4. 延迟 artifact id 校验
- config load 时**不**验证 `rules` 里的 artifact id 是否真实存在。
- 校验延后到 `instruction-loader`（因为那时 schema 已知），详见 `rules-injection` spec。

### 5. 错误处理
- 文件不存在 → 返回 null（不报错）。
- 非法 YAML → 警告 + null。
- Zod 校验失败 → 警告（含细节）+ null。
- 命令在配置解析失败时**仍能跑**（用默认值，如 schema = `spec-driven`）。
- 警告写到 stderr。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 路径 | `openspec/config.yaml`（或 `.yml`） |
| 容错 | 逐字段解析；不因单字段失败而拒绝整文件 |
| 字段 | `schema` / `context` / `rules` |
| context 上限 | 50KB |
| 失败行为 | 返回 null + stderr 警告；命令继续跑默认配置 |
| 校验延迟 | artifact id 校验在 instruction load 时做 |
