# ci-nix-validation

## 目的 (Purpose)

在 CI（GitHub Actions）里**校验 Nix flake 构建与维护脚本**，确保 Nix 用户能稳定地安装和使用 OpenSpec。每次 PR 与 main push 都跑，防止 Nix 支持回退。

## 内容解析 (Content Analysis)

### 1. Nix Flake 构建验证
- 跑 `nix build`，exit 0 且产物含 openspec 二进制 → 通过。
- 失败 → 阻止 PR merge。
- flake 声明多系统时，至少验证 `x86_64-linux`。
- 安装：官方 Nix installer 或 `determinatesystems/nix-installer-action`，带缓存。
- 启用 `experimental-features = nix-command flakes`。
- 5 分钟内完成 clean run，3 分钟内完成缓存 run。

### 2. Update Script 校验
- 跑 `update-flake.sh`：能正确读 `package.json` 版本、验证 `flake.nix` 用动态版本。
- 用 mock hash 验证脚本能检测并提取正确的 pnpm 依赖 hash，写入合法 sha256。

### 3. CI 集成
- 加入 GitHub Actions workflow 并列入**必要检查**。
- 触发：push 到 PR / push 到 main / 手动 `workflow_dispatch`。
- 与其他 CI job（test、lint）并行。

### 4. 本地测试
- 用 `act` 工具本地跑：标准 GitHub Actions 语法，Nix 设置在 act Docker 环境中能工作。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 平台 | GitHub Actions + `act` 本地兼容 |
| 校验项 | `nix build` + `update-flake.sh` |
| 性能预算 | clean ≤5min，cached ≤3min |
| 安装 | determinatesystems/nix-installer-action + cache |
| 触发 | PR、main、手动 |
