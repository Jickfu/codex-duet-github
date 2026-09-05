# 2026-09-05 Review-only Dogfood

这是 `codex-duet-github` 第一次真实 Review 核心闭环 Dogfood。它覆盖真实 GitHub 写入、累计 Commit 评审、问题修复和再次批准，不代表全部角色、平台或远程 CI 门禁已经验证。

本文区分 GitHub 可独立复核的仓库事实与只能由 Executor 根据运行现场报告的会话事实。

## 固定 Skill 版本与测试环境

- 源仓库：[Jickfu/codex-duet-github](https://github.com/Jickfu/codex-duet-github)
- Executor 声明使用的固定 Skill 版本：`b014b61940f718e1c3a3d9cee3f37f68c6780a0a`
- Dogfood 仓库：[Jickfu/codex-duet-github-dogfood](https://github.com/Jickfu/codex-duet-github-dogfood)
- 日期：2026-09-05
- 角色：ChatGPT 仅承担 Review
- 现场环境：Windows 上的 ChatGPT Desktop Codex、Codex 内置 Browser、当前会话正式连接的可写 GitHub 连接器
- 本地目录：`E:\cloud\code\codex-duet-github-dogfood`
- 边界：未创建 Pull Request，未合并，未删除任务分支

GitHub 可以独立验证固定 Skill Commit 存在；实际运行时采用该固定 Skill 副本、桌面和浏览器形态属于 Executor 现场证据。

## 不可变 GitHub 证据

| 角色 | 完整 SHA | 父提交 |
| --- | --- | --- |
| Dogfood 基准 | `e43337280d06fa6ddb4bec1c63d47462debd138a` | `c216b9d77eb068a47a9b7a506c413befcb7b22ea` |
| 首轮 Review 候选 | `516d1a5bb4fad93a008e239c50fbd67f444f8a14` | `e43337280d06fa6ddb4bec1c63d47462debd138a` |
| 修复及最终 Review 候选 | `0c41e5f6e802744792c94cef9dcbe825442e53c4` | `516d1a5bb4fad93a008e239c50fbd67f444f8a14` |

- 任务分支：`codex/first-real-dogfood-20260905`
- 最终分支头：`0c41e5f6e802744792c94cef9dcbe825442e53c4`
- 累计范围：`e43337280d06fa6ddb4bec1c63d47462debd138a..0c41e5f6e802744792c94cef9dcbe825442e53c4`
- GitHub compare：`ahead_by = 2`、`behind_by = 0`、`total_commits = 2`，merge base 等于 Dogfood 基准。

## 首轮变更与预置缺陷

首轮 Commit `516d1a5bb4fad93a008e239c50fbd67f444f8a14` 包含三种文件操作：

- 修改：`README.md`
- 新增：`docs/dogfood-result.md`
- 删除：`docs/delete-me.md`

该 Commit 中，`docs/dogfood-result.md` 的相对链接实际写为不存在的 `./references.md`。GitHub 可以证明错误链接存在；它是为 Reviewer 发现能力测试而预置、且答案没有在 Review 请求中泄露，属于 Executor 现场证据。

## Review 发现、修复与批准

### GitHub 可验证的修复

后续 Commit `0c41e5f6e802744792c94cef9dcbe825442e53c4` 仅修改 `docs/dogfood-result.md`，将 `./references.md` 修正为 `./reference.md`。同一最终 SHA 下，`docs/reference.md` 实际存在，并包含标记 `REFERENCE-TARGET-20260905`。

### Executor 现场观察

以下交互事实来自 Executor 观察到的浏览器会话，不编码在 GitHub Commit 元数据中：

- ChatGPT 首轮实际读取 `e43337280d06fa6ddb4bec1c63d47462debd138a..516d1a5bb4fad93a008e239c50fbd67f444f8a14`，正确列出三个变更路径。
- ChatGPT 发现 `./references.md` 断链，将其列为 `Required correction [Medium]`，并给出目标文件存在、错误路径返回 404 的证据。
- Codex 核实意见成立，修复后重新运行本地检查并发布第二轮 Commit。
- ChatGPT 重新读取完整累计范围 `e43337280d06fa6ddb4bec1c63d47462debd138a..0c41e5f6e802744792c94cef9dcbe825442e53c4`，核实修复并给出 `APPROVED`；Required correction、User decision required 和 Advisory suggestion 均为无。

## 远程 CI 与 Checks

对最终 SHA `0c41e5f6e802744792c94cef9dcbe825442e53c4` 的读取结果：

- Commit Status contexts：0，返回 `statuses: []`
- 关联 workflow runs：0
- 仓库根目录没有 `.github`
- Rulesets：空
- 任务分支可读取的保护摘要显示未启用 required status checks；完整管理级分支保护端点受 GitHub App 权限限制

因此本次验证了按精确 SHA 查询远程检查状态的路径，但没有验证远程 CI 成功门禁或失败门禁，不能表述为“CI 已通过”。

## GitHub 可独立复核的范围

GitHub 可独立复核：

- 固定 Skill Commit、Dogfood 基准和两轮 Commit 均存在；
- `e43337280d06fa6ddb4bec1c63d47462debd138a → 516d1a5bb4fad93a008e239c50fbd67f444f8a14 → 0c41e5f6e802744792c94cef9dcbe825442e53c4` 的线性父子关系；
- 任务分支的最终指针、累计两个 Commit 和三个变更路径；
- 首轮错误链接、第二轮修复及最终目标文件；
- 最终 SHA 没有可观察到的 Status context 或关联 workflow run；
- 任务分支仍存在，且没有以该分支为 head 的 Pull Request。

## 只能由 Executor 报告的现场证据

GitHub 最终状态不能独立证明：

- 运行时采用的是指定固定 Skill 副本；
- Windows、ChatGPT Desktop Codex 和内置 Browser 的具体会话状态；
- 浏览器已登录且 ChatGPT 通过 GitHub 实际读取指定 SHA；
- 预置缺陷的意图及其在 Review 前未被泄露；
- ChatGPT 的自然语言意见、Required correction 分类和最终批准；
- 本地目录身份核对、检查命令及其输出；
- GitHub 写入由当前 Codex 连接器发起，且执行期间未使用本地 `git`、`git.exe` 或 `gh`。

这些事项由 Executor 根据当次运行现场报告；可由最终分支、Commit 和文件状态做部分交叉验证，但证据等级不同。

## 已覆盖与尚未覆盖

已覆盖：

- Review-only 核心闭环；
- 默认分支解析与固定基准 SHA；
- 独立任务分支；
- GitHub 文件新增、修改和删除；
- 两轮真实 Commit 及累计范围 Review；
- Reviewer 发现预置缺陷、修复和再次 Review；
- 精确最终 SHA 的远程 Checks 查询。

尚未覆盖：

- Plan 真实闭环；
- Discussion 真实闭环；
- 非默认 `BASE_REF`；
- 远程 CI 成功门禁；
- 远程 CI 失败门禁；
- macOS 实测。
