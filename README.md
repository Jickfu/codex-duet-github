# codex-duet-github

一个纯指令型 Codex Skill：Codex 负责实施与测试，网页版 ChatGPT 按用户选择参与规划（Plan）、讨论（Discussion）或评审（Review），通过 GitHub 真实仓库和 Commit 协作。

仅明确调用时启用，不自动全选角色。项目不包含通信服务、状态机、后台运行时或测试程序。

当前状态：**Alpha / workflow specification**。结构校验和桌面推演不等于真实端到端验证；GitHub 插件写入及 ChatGPT 精确 Commit 评审闭环尚未完成 Dogfood，不应视为稳定 V1。

## 支持环境

当前版本面向 Windows 和 macOS 的 ChatGPT 桌面应用中的 Codex。这里描述的是目标支持范围，不是已经实测的平台认证。

本次 Codex 会话须同时具备本地完整项目访问、Browser Use 或 Codex Chrome 扩展，以及可写 GitHub 工具连接。Codex CLI、IDE 扩展或没有浏览器通道的环境不属于本版本的完整支持范围。

浏览器能力以当前会话、套餐和工作区策略为准；内置浏览器与 Chrome 使用不同的浏览器状态，只检查选定通道。参见 [官方浏览器说明](https://learn.chatgpt.com/docs/browser?surface=app)。

## 文件与规范

```text
codex-duet-github/
├── README.md
├── LICENSE
├── SKILL.md
└── agents/
    └── openai.yaml
```

仓库根目录就是完整 Skill：[SKILL.md](SKILL.md) 保存流程，[agents/openai.yaml](agents/openai.yaml) 保存展示信息和 `policy.allow_implicit_invocation: false`，仅允许显式调用。README 介绍项目与安装方式，LICENSE 保存 Apache-2.0 许可证全文。

结构依据 [OpenAI 官方 Build skills 文档](https://learn.chatgpt.com/docs/build-skills)（2026-09-04 核对）：Skill 目录必须包含带 `name`、`description` 的 `SKILL.md`，`agents/openai.yaml` 是可选扩展。本项目按需求保留 README，短消息模板直接放在 SKILL 中。

## 安装

将仓库内容（不含 `.git/`）整体复制到以下任一位置，目录名使用 `codex-duet-github`：

- 个人使用：`~/.agents/skills/codex-duet-github/`，Windows 对应用户目录下的 `.agents\skills\codex-duet-github\`。
- 指定仓库使用：`<目标仓库>/.agents/skills/codex-duet-github/`。

安装后直接存在 `codex-duet-github/SKILL.md`、`codex-duet-github/agents/openai.yaml` 和 `codex-duet-github/LICENSE`。也可请 `$skill-installer` 从本仓库根目录安装，并指定名称 `codex-duet-github`。

可从已经发布的任务分支或合并后的默认分支下载文件；未合并时不要假设默认分支已有完整 Skill。若同名安装已经存在，先检查内容，再决定更新，不直接覆盖。安装副本的后续更新需要同步复制。

Codex 会自动发现变化；若没有显示，重启 Codex。在 Skill 选择器中选中它，或输入 `$codex-duet-github` 明确调用。仅把源项目放在普通代码目录下不等于安装。本项目不自动修改个人 Codex 配置。

## 前置条件

- Codex 已连接当前环境提供的 GitHub 插件，账户有目标仓库读写权限。
- 插件能读取默认分支/完整 SHA、创建分支、发布 Commit 并读取验证结果；“连接成功”不保证具备所有写能力。
- 本地完整目标项目可读取、可写入，所需构建/测试工具可执行。
- Codex 有可用的 Browser / Computer Use 通道，本次选定浏览器已登录网页版 ChatGPT。
- 本次实际使用的 ChatGPT 对话界面能够调用 GitHub，并获得目标仓库和指定 Commit 的读取权限。这与 Codex 的 GitHub 连接是两项独立检查；ChatGPT 能读取仓库不证明 Codex 连接具有写能力。

只检查实际使用的路径。例如选定内置浏览器时，不要求 Chrome 同时登录。首次运行将通过真实消息和已知 Commit 文件核对 ChatGPT 的仓库访问。

目标仓库须先创建、初始化并授权。所有 GitHub 仓库操作都走 Codex GitHub 插件，不自动用本地 Git、gh 或网页编辑替代。

## 使用示例

明确调用后附上目标仓库、本地目录、具体目标和验收标准。支持以下四种组合：

```text
使用 codex-duet-github，ChatGPT只负责Review，完成以下任务：……
使用 codex-duet-github，启用Plan和Review，完成以下任务：……
使用 codex-duet-github，只启用Discussion，完成以下任务：……
使用 codex-duet-github，ChatGPT承担Plan、Discussion和Review，完成以下任务：……
```

也可以使用中文角色或显式 Skill 提及：

```text
使用 $codex-duet-github，ChatGPT只负责代码审查，不参与规划。
目标仓库：owner/repository
本地目录：目标项目的实际路径
任务：修复取消操作后仍生成输出的问题。
验收：取消后没有新输出；补充回归测试，现有测试通过。

使用 codex-duet-github，启用规划、技术讨论和评审，完成以下任务：……
```

缺少角色时只询问一次：“本任务需要ChatGPT参与哪些角色：Plan、Discussion、Review？可以选择一个或多个。”角色明确前不改文件、不建分支、不发送任务内容。

## 工作流程

明确角色 → 实际启动检查 → 固定默认分支 `BASE_SHA` 并创建独立任务分支 → 所选 Plan → Codex 实施与所选 Discussion → 测试 → 插件发布 Commit 并验证 SHA → 所选 Review 与修复闭环。

Review 检查 `BASE_SHA..最新 REVIEW_SHA` 的累计修改。只选 Review 不会自动要求 Plan；只选 Discussion 仍由 Codex 自行规划、实施、验证与发布，并实际讨论关键方案。重要分歧无法达成共识时请用户决策。完成时报告任务分支和最终 SHA；仅在用户明确要求时合并或删除分支，不固定追问。

远程任务分支不代表本地目录已切换分支。首次修改每个文件前，核对本地内容与 `BASE_SHA`，维护新增、修改、删除清单；已有差异未经同意不覆盖、不发布。发布只包含清单内变更。触及文件校验不保证整个测试环境一致：存在影响测试的本地差异或无法确认等价性时，只报告本地工作区验证，不宣称最终 Commit 已获得精确测试。

ChatGPT 环境检查使用指定 SHA、指定文件及不预先透露答案的内容校验；正式 Review 分别确认基准、评审 Commit 和累计差异可读。仅回显 SHA 或命中多个版本共有的内容不能独立证明精确读取。

## 首版验证

纯指令项目采用结构校验与场景桌面推演，不开发额外测试程序。下表是对 SKILL 指令分支的人工推演结果，**不是已完成真实浏览器会话的端到端测试**。

| 场景 | 推演结果 |
| --- | --- |
| 明确调用但未选择角色 | 先询问一次并等待；不修改、不建分支、不联系 ChatGPT |
| Codex GitHub 未连接 | 实际调用失败后提示连接插件，暂停正式任务 |
| 选定浏览器未登录 | 请用户在该浏览器手动登录，随后重新检查 |
| ChatGPT 不能读取目标仓库/Commit | 请用户连接 GitHub、授权目标仓库并重查；不接受猜测或粘贴代码替代 |
| 只选 Review / 只做代码审查 | Codex 自行规划；测试与发布后请求 Review，不请求 Plan |
| 选择 Plan + Discussion + Review / 规划、讨论、评审 | 先取得计划；关键问题先独立思考再讨论；实施、发布、真实 Commit 评审及修复闭环 |
| 插件缺少必要写操作 | 停止依赖发布的流程，明确未发布；不虚构成功或改用 Git |
| 只选 Discussion | 实际讨论关键方案，Codex 实施验证发布，不自动启动 Plan/Review |
| 仅介绍或编辑本 Skill | 不启动它所描述的协作流程 |
| Review 后修复产生新 Commit | 重新测试、发布、读取验证 SHA，再评审累计范围 |

安装到实际环境后，真实浏览器登录、消息发送、ChatGPT 的 GitHub 读取和插件写入仍需在启动/发布时验证。最终交付报告须区分结构校验、人工推演和真实执行结果。

### V1 前的真实 Dogfood（待执行）

在用户授权的测试仓库中完成并记录证据，不能用桌面推演勾选：

- [ ] Codex 通过 GitHub 插件读取账户、目标仓库、默认分支和完整 `BASE_SHA`。
- [ ] 从确切 `BASE_SHA` 创建任务分支，并重新读取验证分支 SHA。
- [ ] 校验触及文件，修改一个文档文件并运行本地检查，注明测试范围。
- [ ] 通过插件发布首个 Commit，并验证分支 SHA、父提交及实际内容。
- [ ] ChatGPT 从 GitHub 读取精确 `REVIEW_SHA`，完成内容校验并提出实际修改建议。
- [ ] Codex 核实建议、修复、重测并发布第二个 Commit，重新验证远端结果。
- [ ] ChatGPT 读取并评审累计 `BASE_SHA..REVIEW_SHA_2`，完成修复闭环。
- [ ] 在同次测试中覆盖多文件提交及文件删除，验证其他文件保持不变。

记录日期、桌面系统/产品形态、连接器及实际写动作、仓库/分支、完整 SHA、检查命令与结果、ChatGPT 对话证据和读取限制。发布结果不明、分支未更新或指定 Commit 不可读都不能记为通过。

完成后才可标记 `Tested on ChatGPT Desktop Codex + write-capable GitHub connector`，并附实际环境与证据；这不是对所有 GitHub 连接器或所有平台的能力承诺。当前尚无此认证。

## 许可证

本项目采用 [Apache License 2.0](LICENSE)（SPDX：`Apache-2.0`）。
