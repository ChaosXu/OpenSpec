# cli-completion

## 目的 (Purpose)

为 OpenSpec CLI 提供**多 shell 兼容**的 tab 补全脚本，支持 Zsh、Bash、Fish、PowerShell。涵盖静态（命令、flag）与动态（change id、spec id）补全，含安装、卸载、生成子命令。目标是让 OpenSpec 在任何主流 shell 下都用起来像"原生"。

## 内容解析 (Content Analysis)

### 1. Shell 原生体验
- **Zsh**：用 `_arguments` / `_describe` / `compadd`，单 TAB 触发菜单（Oh My Zsh 增强样式自动识别）。
- **Bash**：`complete` + `COMPREPLY`，双 TAB 触发，支持 bash-completion v1/v2。
- **Fish**：`complete -c openspec` + 条件，单 TAB + 自动建议预览，依赖 Fish 内建缓存。
- **PowerShell**：`Register-ArgumentCompleter` + scriptblock，循环切换，支持 Windows PowerShell 5.1 与 PowerShell Core 7+。
- **明确禁止**自定义触发键或导航方式（"feel native"）。

### 2. 子命令结构
`openspec completion {generate|install|uninstall} [shell]`，shell 不传时从环境变量自动检测。

### 3. Shell 检测
- 读 `$SHELL` 路径，提取 `zsh` / `bash` / `fish` / `powershell`。
- 找不到 / 不支持 → 报错列出受支持的 shell。
- PowerShell 通过 `$PSModulePath` 存在性识别。

### 4. 生成（generate）
- 输出到 stdout 完整的补全脚本（可重定向到文件）。
- 包含 init / list / show / validate / archive / view / update / change / spec / completion 等所有命令的 flag。
- 动态值用 `openspec __complete` 内部命令提供（shell 通过子进程拉取）。

### 5. 动态补全
- change id 来自 `openspec/changes/`（排除 archive/）。
- spec id 来自 `openspec/specs/`。
- **缓存 2 秒**，过期自动重读。
- 不在 OpenSpec 项目里时只补静态内容（命令 + flag），不查文件系统。

### 6. 安装（install）
- **Zsh**：检测 Oh My Zsh → 写 `~/.oh-my-zsh/custom/completions/_openspec`；否则写 `~/.zsh/completions/_openspec` 并在 `~/.zshrc` 加 `fpath` + `compinit`。
- **Bash**：检测 bash-completion → 写 `/etc/bash_completion.d/openspec`（sudo）或 `~/.local/share/bash-completion/completions/openspec`；否则写 `~/.bash_completion.d/openspec` 并在 `~/.bashrc` 加 source。
- **Fish**：写 `~/.config/fish/completions/openspec.fish`（Fish 自动加载，无需改 config）。
- **PowerShell**：找到 profile 位置（`$PROFILE` 或默认路径），用 marker-based 更新追加 import。
- 重复安装时：提示"已安装"，允许覆盖；exit 0。

### 7. 卸载（uninstall）
- 默认需 `--yes` 才能跳过确认提示。
- 各 shell 用 marker-based 移除 fpath / source 行 / profile import。
- 未安装时：报错 + exit 1。

### 8. 架构
- 工厂模式：`CompletionFactory.createGenerator(shell)` / `createInstaller(shell)`。
- 每 shell 一个 Generator（`ZshGenerator` / `BashGenerator` / `FishGenerator` / `PowerShellGenerator`）。
- 每 shell 一个 Installer（同上）。
- 命令注册中心 `COMMAND_REGISTRY`（`CommandDefinition[]`）作为唯一真相源——所有 generator 消费同一份数据。
- `SupportedShell = 'zsh' | 'bash' | 'fish' | 'powershell'` 字面量类型 + TypeScript exhaustiveness。
- `CompletionProvider` 封装 `getChangeIds` / `getSpecIds` / `isOpenSpecProject`，2 秒 TTL 缓存。

### 9. 错误处理
- 不支持的 shell：明确报错 + 列出受支持列表。
- 权限不足：报错 + 建议 sudo 或替代路径。
- 缺 config 目录：自动创建 + 通知用户。
- 缺 `$SHELL` 无法自动检测：明确提示传 shell 名。

### 10. 输出
- generate：纯脚本到 stdout，可重定向。
- install/uninstall：彩色（除非 `--no-color`）✓ 标记 + 路径 + reload 提示。
- `--verbose`：逐步显示 shell 检测、文件路径、配置改动、创建确认。

### 11. 测试
- 用 mock 替换 `$SHELL` / `$PSModulePath`，通过依赖注入 FS 操作。
- 4 个 shell 各有独立 generator / installer 测试套件（用临时目录、FS mock）。
- 跨 shell 一致性：命令 / flag / 动态值 / 错误信息全部对齐。

## 核心要点总结

| 维度 | 关键约束 |
|------|----------|
| 支持的 shell | zsh / bash / fish / powershell |
| 子命令 | `generate` / `install` / `uninstall` [shell] |
| 动态补全 | change id、spec id（2s 缓存） |
| 安装路径 | 各 shell 原生约定（Oh My Zsh、bash-completion、fish conf dir、PS profile） |
| 架构 | 工厂 + 注册中心 + 4 个 generator/installer |
| UX | 严格遵守各 shell 原生触发键与导航 |
