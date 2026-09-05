# codex-duet-github

一个纯指令型 Codex Skill：Codex 负责实施与测试，网页版 ChatGPT 按用户选择参与规划（Plan）、讨论（Discussion）或评审（Review），通过 GitHub 真实仓库和 Commit 协作。

仅明确调用时启用，不自动全选角色。项目不包含通信服务、状态机、后台运行时或测试程序。

当前状态：**Beta — Review 核心闭环已完成首次真实 Dogfood，Plan + Review 已在一次自举文档任务中执行**。这不等于稳定 V1，也不证明复杂代码任务中的 Plan 质量；Discussion、非默认 `BASE_REF`、远程 CI 成功/失败门禁和 macOS 实测仍未完成。详见 [Review-only Dogfood 证据](docs/dogfood/2026-09-05-review-only.md)和 [Plan + Review 自举 Dogfood 证据](docs/dogfood/2026-09-05-plan-review-self-hosted.md)。

## 支持环境

当前版本面向 Windows 和 macOS 的 ChatGPT 桌面应用中的 Codex。这里描述的是目标支持范围，不是已经实测的平台认证。

本次 Codex 会话须同时具备本地完整项目访问、ChatGPT 内置 Browser 或已连接的 ChatGPT 浏览器扩展，以及可写 GitHub 工具连接。Codex CLI、IDE 扩展或没有浏览器通道的环境不属于本版本的完整支持范围。

浏览器能力以当前会话、套餐和工作区策略为准；内置浏览器与 Chrome 使用不同的浏览器状态，只检查选定通道。参见 [官方浏览器说明](https://learn.chatgpt.com/docs/browser?surface=app)。

[ChatGPT 浏览器扩展](https://learn.chatgpt.com/docs/chrome-extension) 可连接 Chrome、Edge、Brave、Opera 和 Vivaldi，实际可用性以当前会话为准。

## 文件与规范

```text
codex-duet-github/
├── README.md
├── LICENSE
├── SKILL.md
├── agents/
│   └── openai.yaml
└── docs/
    └── dogfood/
        ├── 2026-09-05-plan-review-self-hosted.md
        └── 2026-09-05-review-only.md
```

仓库根目录就是完整 Skill：[SKILL.md](SKILL.md) 保存流程，[agents/openai.yaml](agents/openai.yaml) 保存展示信息和 `policy.allow_implicit_invocation: false`，仅允许显式调用。README 介绍项目与安装方式，LICENSE 保存 Apache-2.0 许可证全文。

结构依据 [OpenAI 官方 Build skills 文档](https://learn.chatgpt.com/docs/build-skills)（2026-09-04 核对）：Skill 目录必须包含带 `name`、`description` 的 `SKILL.md`，`agents/openai.yaml` 是可选扩展。本项目按需求保留 README，短消息模板直接放在 SKILL 中。

## 安装

将仓库内容（不含 `.git/`）整体复制到以下任一位置，目录名使用 `codex-duet-github`：

- 个人使用：`~/.agents/skills/codex-duet-github/`，Windows 对应用户目录下的 `.agents\skills\codex-duet-github\`。
- 指定仓库使用：`<目标仓库>/.agents/skills/codex-duet-github/`。

安装后直接存在 `codex-duet-github/SKILL.md`、`codex-duet-github/agents/openai.yaml` 和 `codex-duet-github/LICENSE`。也可向安装器提供：

```text
使用 $skill-installer 安装以下 Skill：
Repository: Jickfu/codex-duet-github
Path: .
Name: codex-duet-github
```

手动安装使用上述公开文档推荐的 `.agents/skills` 位置；使用 `$skill-installer` 时，由当前 Codex 版本的安装器选择其支持的目录。当前安装器默认使用 `$CODEX_HOME/skills`（通常为 `~/.codex/skills`），不保证与手动目录相同。安装后在下一轮通过 Skill 选择器、`/skills` 或 `$codex-duet-github` 验证是否已发现。

可从已经发布的任务分支或合并后的默认分支下载文件；未合并时不要假设默认分支已有完整 Skill。若同名安装已经存在，先检查内容，再决定更新，不直接覆盖。安装副本的后续更新需要同步复制。

Codex 会自动发现变化；若没有显示，重启 Codex。在 Skill 选择器中选中它，或输入 `$codex-duet-github` 明确调用。仅把源项目放在普通代码目录下不等于安装。本项目不自动修改个人 Codex 配置。

## 前置条件

- Codex 已连接当前环境提供的 GitHub 插件，账户有目标仓库读写权限。
- 插件能解析基准引用/完整 SHA、创建分支、发布 Commit 并读取验证结果；“连接成功”不保证具备所有写能力。
- 账户身份和权限元数据接口不是硬性要求；不提供时分别报告不可观察，并以实际读取、分支创建和发布验证连接与能力，不把仓库 owner 当作连接账户。
- 本地完整目标项目可读取、可写入，所需构建/测试工具可执行。
- Codex 有可用的 Browser / Computer Use 通道，本次选定浏览器已登录网页版 ChatGPT。
- 本次实际使用的 ChatGPT 对话界面能够调用 GitHub，并获得目标仓库和指定 Commit 的读取权限。这与 Codex 的 GitHub 连接是两项独立检查；ChatGPT 能读取仓库不证明 Codex 连接具有写能力。

只检查实际使用的路径。例如选定内置浏览器时，不要求 Chrome 同时登录。首次运行将通过真实消息和已知 Commit 文件核对 ChatGPT 的仓库访问。

目标仓库须先创建、初始化并授权。用户可指定基准分支、标签或完整 Commit；未指定时才使用默认分支。所有 GitHub 仓库操作都走当前会话正式连接的 GitHub 插件、连接器或 GitHub 工具；工具以 GitHub API URL 为参数仍是合法插件操作。禁止自行用本地 Git、gh、通用网络客户端或网页编辑绕过插件。

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

明确角色与 `BASE_REF` → 实际启动检查并解析 `BASE_SHA` → 从确切 `BASE_SHA` 创建独立任务分支并固定基准 → 所选 Plan → Codex 实施与所选 Discussion → 测试 → 插件发布 Commit 并验证 SHA → 所选 Review 与修复闭环。

启动报告分别列明写入动作是否可用、声明权限是否可观察，以及分支创建和 Commit 发布是否已实测；发现工具不代表写权限已生效。创建分支前检查名称冲突，已有同名分支默认换唯一后缀；仅在用户明确恢复任务且核对身份与历史后复用，不重置已有分支。

每次发布后读取精确 `REVIEW_SHA` 的远程 CI / Checks，报告检查名称、状态和读取限制。必需检查失败须核实处理，进行中、取消或未运行不算通过；未满足必需检查不能宣布验收完成。区分“未配置 CI”和“检查状态不可观察”，不把本地测试当作远程 CI 证据。

正式 Review 同时提供整体与本阶段验收标准，以及此前已验收阶段及其 SHA。意见分为 Required correction（必须修正）、User decision required（需要用户决定）和 Advisory suggestion（可选建议）；严重程度不直接决定是否必须修复。前两类须解决才能完成，可选建议可以保留并在最终报告说明。

`BASE_REF` 由用户指定时优先使用分支、标签或完整 Commit，未指定时使用默认分支；解析后的 `BASE_SHA` 在任务分支创建后固定。Review 检查 `BASE_SHA..最新 REVIEW_SHA` 的累计修改。只选 Review 不会自动要求 Plan；只选 Discussion 仍由 Codex 自行规划、实施、验证与发布，并实际讨论关键方案。重要分歧无法达成共识时请用户决策。完成时报告 `BASE_REF`、`BASE_SHA`、任务分支和最终 SHA；仅在用户明确要求时合并或删除分支，不固定追问。

远程任务分支不代表本地目录已切换分支。首次修改每个文件前，核对本地内容与 `BASE_SHA`，维护新增、修改、删除清单；已有差异未经同意不覆盖、不发布。发布只包含清单内变更。触及文件校验不保证整个测试环境一致：存在影响测试的本地差异或无法确认等价性时，只报告本地工作区验证，不宣称最终 Commit 已获得精确测试。

启动时先用远端根目录清单和 1 至 3 个项目标识文件核对本地项目身份，无法确认时请用户确认路径。身份检查不要求整个工作区等于基准。文本比较区分业务修改与 LF/CRLF、UTF-8 BOM 表示差异，发布时避免无意的整文件转换。

优先多文件单 Commit；插件仅支持逐文件写入时，允许连续 Commit（删除也可单独提交）。每次写入前后核对分支及 Commit，全部变更发布完成才以最终 SHA 请求 Review；中途失败报告已发布 SHA、当前分支及剩余文件，不宣称阶段完成。

ChatGPT 环境检查使用指定 SHA、指定文件及不预先透露答案的内容校验。启用 Review 且有合适历史范围时，追加已知小范围读取检查；否则分别报告单 Commit 读取状态和“累计差异读取：待首次正式 Review 验证”。正式 Review 要求实际变更路径、关键实现事实、精确版本及读取限制，并由 Codex 对照插件结果核实；仅回显 SHA 不算证据。“未发现需要修改的问题”是合法结论。

已有本地修改获准纳入任务且实质改变接口、架构、数据、范围或测试策略时，应更新任务事实：已选择 Plan 则重新检查受影响计划，已选择 Discussion 则按重要性讨论；否则由 Codex 重新规划。对存在可验证历史范围但 ChatGPT 读取失败的 Review 通道，应在实施前暂停；若用户要求先实现，可以保留实现与已发布 Commit，但不得宣布任务完成，直到 Review 恢复或用户明确取消该角色。

## 首版验证

纯指令项目采用结构校验与场景桌面推演，不开发额外测试程序。下表仍只是对 SKILL 指令分支的人工推演结果；真实执行证据见 [Review-only Dogfood](docs/dogfood/2026-09-05-review-only.md)和 [Plan + Review 自举 Dogfood](docs/dogfood/2026-09-05-plan-review-self-hosted.md)。

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
| 仅有逐文件写操作，第二次写入失败 | 保留并报告首个已验证 SHA、当前分支与剩余文件，不启动 Review |
| 本地目录属于另一项目 | 项目标识核对失败，在规划/讨论前暂停并请用户确认路径 |
| 文本仅有 CRLF 或 UTF-8 BOM 差异 | 标为表示差异，发布前检查没有无意转换 |
| 没有账户身份接口 / 没有合适历史范围 | 分别报告身份不可用 / 累计读取待验证，不等同于连接失败 |
| 权限元数据不可观察 | 记录限制，以实际读取、分支创建和发布验证能力 |
| 用户指定 `develop`、`release/1.x` 或完整 Commit | 将该 `BASE_REF` 解析为 `BASE_SHA`，从其创建任务分支 |
| 历史范围可读但 ChatGPT 读取失败 | 在实施前暂停；用户可修复授权、移除 Review，或明确要求仅先实现 |
| Review 仅回显 SHA 或只读部分文件 | 不接受为完整范围通过，要求补足可核对的事实证据 |
| Reviewer 未发现预置缺陷 | 发现能力不记通过，不要求虚构其他问题 |

每次安装到实际环境后，仍须在启动/发布时验证真实浏览器登录、消息发送、ChatGPT 的 GitHub 读取和插件写入。最终交付报告须区分结构校验、人工推演和真实执行结果。

### V1 前的真实 Dogfood

Review-only 首次真实 Dogfood 已完成，以下项目均有 [固定证据](docs/dogfood/2026-09-05-review-only.md)：

- [x] 将默认分支 `BASE_REF` 解析为固定 `BASE_SHA`，并核对账户、权限和目标仓库。
- [x] 核对本地目录身份，完成单 Commit 与累计范围的 ChatGPT GitHub 读取检查。
- [x] 从固定 `BASE_SHA` 创建并验证独立任务分支。
- [x] 校验触及文件，完成多文件新增、修改、删除及本地检查。
- [x] 在 Reviewer 不可见的控制信息中预置可客观验证的错误链接。
- [x] 通过 GitHub 连接发布首轮 Commit，并验证分支、父提交、路径与内容。
- [x] ChatGPT 读取精确首轮 `REVIEW_SHA` 和累计范围，发现预置问题。
- [x] Codex 核实、修复、重测并发布第二轮 Commit。
- [x] ChatGPT 重新读取并批准 `BASE_SHA..REVIEW_SHA_2` 累计范围。

另一次[自举文档任务](docs/dogfood/2026-09-05-plan-review-self-hosted.md)已真实执行 ChatGPT Plan、Codex 核对与实施、GitHub 发布以及累计 Review。它证明 Plan + Review 角色流程可在该范围内走通，但不证明复杂代码任务中的 Plan 质量。

Discussion、非默认 `BASE_REF`、远程 CI 成功/失败门禁和 macOS 实测仍未完成。

若 ChatGPT 未发现预置问题，本轮 Review 发现能力及修复闭环不记为通过，不要求它虚构其他问题。若没有预置缺陷，“未发现问题”是合法结果，但该场景不能证明 Review 修复闭环。读取能力、发现能力和修复闭环分别记录结果。

记录日期、桌面系统/产品形态、连接器及实际写动作、仓库/分支、完整 SHA、检查命令与结果、ChatGPT 对话证据和读取限制。发布结果不明、分支未更新或指定 Commit 不可读都不能记为通过。

Review-only 路径可标记 `Tested on ChatGPT Desktop Codex + write-capable GitHub connector`，但必须附实际环境与 [证据范围](docs/dogfood/2026-09-05-review-only.md)；这不是对所有角色、GitHub 连接器或平台的能力承诺。

## 许可证

本项目采用 [Apache License 2.0](LICENSE)（SPDX：`Apache-2.0`）。
