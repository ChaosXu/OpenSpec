# Spec 列表（按首次入库时间升序）

> 数据来源：`git log --diff-filter=A --name-only --pretty=format:"%ad|%s" --date=short -- openspec/specs/`
> "首次入库时间"反映该 spec 首次出现在仓库的提交日期，而非内容最后修订时间。

| # | 日期 | Spec | 来源 commit 主题 |
|---|------|------|------------------|
| 1 | 2025-08-06 | `openspec-conventions` | adopt future state storage for OpenSpec changes |
| 2 | 2025-08-07 | `cli-init` | archive completed init command change |
| 3 | 2025-08-11 | `cli-update` | archive add-update-command change |
| 4 | 2025-08-13 | `cli-list` | archive add-list-command change |
| 5 | 2025-08-13 | `cli-diff` | archive diff command *(已被 remove-diff-command 删除)* |
| 6 | 2025-08-13 | `cli-archive` | archive add-archive-command change |
| 7 | 2025-08-20 | `cli-change` | archive adopt-delta-based-changes / add-zod-validation / adopt-verb-noun-cli-structure |
| 8 | 2025-08-20 | `cli-show` | 同上 |
| 9 | 2025-08-20 | `cli-spec` | 同上 |
| 10 | 2025-08-20 | `cli-validate` | 同上 |
| 11 | 2025-09-12 | `cli-view` | add openspec view dashboard command |
| 12 | 2025-10-14 | `docs-agent-instructions` | archive 4 completed changes and update specs |
| 13 | 2025-12-12 | `cli-completion` | oh-my-zsh-completions |
| 14 | 2025-12-20 | `global-config` | implement global config directory with XDG support |
| 15 | 2025-12-22 | `cli-config` | add openspec config command |
| 16 | 2025-12-25 | `artifact-graph` | add artifact graph core query system |
| 17 | 2025-12-26 | `change-creation` | add change creation utilities |
| 18 | 2025-12-28 | `instruction-loader` | add instruction loader |
| 19 | 2025-12-29 | `cli-artifact-workflow` | add artifact workflow CLI commands (Slice 4) |
| 20 | 2026-01-06 | `specs-sync-skill` | add /opsx:sync command |
| 21 | 2026-01-06 | `opsx-archive-skill` | archive completed changes |
| 22 | 2026-01-09 | `telemetry` | add optional anonymous usage statistics |
| 23 | 2026-01-16 | `ci-nix-validation` | add nix flake support |
| 24 | 2026-02-16 | `ai-tool-paths` | bulk archive completed changes |
| 25 | 2026-02-16 | `cli-feedback` | 同上 |
| 26 | 2026-02-16 | `command-generation` | 同上 |
| 27 | 2026-02-16 | `config-loading` | 同上 |
| 28 | 2026-02-16 | `context-injection` | 同上 |
| 29 | 2026-02-16 | `legacy-cleanup` | 同上 |
| 30 | 2026-02-16 | `opsx-onboard-skill` | 同上 |
| 31 | 2026-02-16 | `opsx-verify-skill` | 同上 |
| 32 | 2026-02-16 | `rules-injection` | 同上 |
| 33 | 2026-02-16 | `schema-fork-command` | 同上 |
| 34 | 2026-02-16 | `schema-init-command` | 同上 |
| 35 | 2026-02-16 | `schema-resolution` | 同上 |
| 36 | 2026-02-16 | `schema-validate-command` | 同上 |
| 37 | 2026-02-16 | `schema-which-command` | 同上 |
| 38 | 2026-05-04 | `workspace-foundation` | archive workspace foundation |
| 39 | 2026-05-06 | `workspace-links` | archive workspace create and register repos |
| 40 | 2026-05-06 | `workspace-open` | propose workspace open agent context |
| 41 | 2026-05-15 | `workspace-change-planning` | add workspace change planning workflow |

## 备注

- 合计 40 个已部署的 spec（`openspec/specs/` 当前存在 40 个目录）。
- `cli-diff` 已通过 `remove-diff-command` 删除，但 `git log --diff-filter=A` 仍记录其首次入库日 2025-08-13，因此以脚注形式保留作为历史记录。
- 2025-08-20 的 `cli-change` / `cli-show` / `cli-spec` / `cli-validate` 四个 spec 在同一 commit 内批量定型；它们的 ADDED 标注直到 2026-02-16 的批量归档才正式写入。
- 活跃阶段（尚未归档）的变更位于 `openspec/changes/`，不在本表范围内。

## 当前 `specs/` 目录 vs 表内统计

- 表内 41 条记录，其中 `cli-diff`（#5）已被删除。
- 表内其余 40 条与 `openspec/specs/` 目录实际内容一一对应。