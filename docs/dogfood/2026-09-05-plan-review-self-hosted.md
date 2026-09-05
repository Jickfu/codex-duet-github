# 2026-09-05 Plan + Review 自举 Dogfood

本次运行使用 `codex-duet-github` 规划并评审其自身的首次 Dogfood 证据固化与 Beta 状态升级，是一次真实的 Plan + Review 自举文档任务。它证明该角色流程在小型文档任务中可以走通，不证明复杂代码任务中的 Plan 质量，也不代表稳定 V1。

本文区分 GitHub 可独立复核的仓库事实与只能由 Executor 根据当次运行现场报告的 Plan、Browser 和本地验证事实。

## 固定版本与运行范围

- 仓库：[Jickfu/codex-duet-github](https://github.com/Jickfu/codex-duet-github)
- 本次证据固化任务使用的固定 Beta Skill：`f8a852302842786a333ed7772f39abb778d6ed54`
- 被记录的自举运行基准：`b014b61940f718e1c3a3d9cee3f37f68c6780a0a`
- 被记录的自举任务分支：`codex/dogfood-evidence-beta-20260905`
- 被记录的最终 Review SHA：`f8a852302842786a333ed7772f39abb778d6ed54`
- 被记录的累计范围：`b014b61940f718e1c3a3d9cee3f37f68c6780a0a..f8a852302842786a333ed7772f39abb778d6ed54`
- 现场环境：Windows 上的 ChatGPT Desktop Codex、可控 Browser 和当前会话正式连接的 GitHub 工具
- 边界：未创建 Pull Request，未删除任务分支；批准 Commit 后来以非强制 fast-forward 发布到 `main`

固定 Commit、分支和最终仓库内容可由 GitHub 复核；实际加载该固定 Skill 的运行时状态、Browser 会话及工具使用方式属于 Executor 现场证据。

## GitHub 可独立验证的结果

GitHub 可重新验证：

- `f8a852302842786a333ed7772f39abb778d6ed54` 的唯一父提交为 `b014b61940f718e1c3a3d9cee3f37f68c6780a0a`；
- compare 状态为 `ahead`，`ahead_by = 1`、`behind_by = 0`、`total_commits = 1`，merge base 等于基准；
- 累计范围只包含三个变更路径：
  - 修改 `README.md`；
  - 修改 `SKILL.md`；
  - 新增 `docs/dogfood/2026-09-05-review-only.md`；
- `codex/dogfood-evidence-beta-20260905` 仍指向最终 Review SHA；
- `main` 后来 fast-forward 到同一 SHA，没有额外 Merge Commit；
- 没有以该任务分支为 head 的 Pull Request；
- 精确最终 SHA 没有可观察到的 Commit Status context 或关联 workflow run，Rulesets 为空。

这些事实证明实际发布内容、线性历史和最终引用状态，但不能单独证明浏览器中的 Plan/Review 对话、本地检查或 ref 更新调用的 `force=false` 参数。

## ChatGPT Plan 的主要结论

以下内容由 Executor 从当时仍可访问的 ChatGPT 对话整理，属于现场会话证据：

- 以单独的 Review-only Dogfood 文档作为完整证据源，README 和 SKILL 只保留状态摘要与链接；
- 严格区分 GitHub 可验证事实与 Executor 现场声明，尤其不能让 Commit 元数据替代 Browser、自然语言 Review 或固定安装副本证据；
- README 只升级到 Beta，并明确 Plan、Discussion、非默认 `BASE_REF`、远程 CI 成功/失败门禁及 macOS 的未覆盖边界；
- SKILL 只修正过时 Alpha 状态，不复制完整报告、不改变既有流程规则；
- `agents/openai.yaml` 不需要修改，也不应增加通信协议、状态文件、脚本或运行时；
- 文档验证应覆盖相对链接、完整 SHA、过时措辞、Skill 结构和严格的三文件发布范围；
- 空 Status/Checks 只能说明没有可观察到的远程运行，不能写成“CI 已通过”。

## Codex 对计划的核对与实施

以下过程只能由 Executor 根据运行现场报告：

1. Codex 先通过 GitHub 独立读取源项目和 Dogfood 仓库，核实基准、两轮 Dogfood Commit、父子关系、错误链接、修复结果及空 Checks，与 Plan 使用的仓库事实一致。
2. Codex 接受“单一证据源”“证据等级分离”“SKILL 最小修改”和“无新增运行时”的方向；同时将 README 的文件树与 Dogfood 状态清单同步到新证据入口，以避免导航和状态继续过时。
3. 实际实施严格限于 `README.md`、`SKILL.md` 和新增 Review-only 证据文档，`agents/openai.yaml` 与 Dogfood 仓库保持不变。
4. Codex 运行 Skill 结构、Markdown 相对链接、尾随空白、围栏、最终换行、完整 SHA、旧状态措辞和文件集合检查；发布后再核对分支、父提交、累计路径和精确 SHA 的远程状态。
5. Codex 仅通过正式 GitHub 工具创建任务分支和发布单 Commit，没有使用本地 `git`、`git.exe` 或 `gh`。

GitHub 可交叉验证第 1、3、5 项的最终仓库结果，但不能独立重放这些本地命令、判断过程和工具调用现场。

## 最终 Review

ChatGPT 在同一专用对话中通过 GitHub 读取指定的 `BASE_SHA`、`REVIEW_SHA` 和完整累计范围，正确列出三个变更路径，并分别核实 README 的 Beta 边界、SKILL 的克制修改、证据文档的事实链与证据等级区分。

Executor 记录的最终结论为：

- `Required correction`：无；
- `User decision required`：无；
- `Advisory suggestion`：无；
- 总体结论：`APPROVED`。

ChatGPT 的自然语言结论与实际读取行为属于 Browser 现场证据；GitHub 只能独立验证它所评审的提交和内容确实存在。

## 覆盖结论与剩余边界

本次可记录为：Plan + Review 已在一次真实自举文档任务中执行，并完成从计划、Codex 核对、实施、验证、GitHub 发布到累计 Review 的闭环。

尚未验证：

- 复杂代码任务中的 Plan 质量；
- Discussion 真实闭环；
- 非默认 `BASE_REF`；
- 远程 CI 失败门禁；
- 远程 CI 成功门禁；
- macOS 实测。
