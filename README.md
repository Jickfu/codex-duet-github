# codex-duet-github

一个纯指令型 Codex Skill：Codex 负责实施与测试，网页版 ChatGPT 按用户选择参与规划（Plan）、讨论（Discussion）或评审（Review），通过 GitHub 真实仓库和 Commit 协作。

仅明确调用时启用，不自动全选角色。项目不包含通信服务、状态机、后台运行时或测试程序。

## 文件与规范

```text
codex-duet-github/
├── README.md
├── LICENSE
└── skill/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

`skill/` 是可安装的 Skill 内容目录：`skill/SKILL.md` 保存完整流程，`skill/agents/openai.yaml` 保存展示信息和 `policy.allow_implicit_invocation: false`。该设置关闭隐式调用，仍允许显式选择 Skill。根目录的 `README.md` 介绍项目与安装方式，`LICENSE` 保存 Apache-2.0 许可证全文。

结构依据 [OpenAI 官方 Build skills 文档](https://learn.chatgpt.com/docs/build-skills)（2026-09-04 核对）：Skill 目录必须包含带 `name`、`description` 的 `SKILL.md`，`agents/openai.yaml` 是可选扩展。本项目按需求保留 README，短消息模板直接放在 SKILL 中。

## 安装

将 `skill/` 内的内容复制到以下任一位置，安装目录名使用 `codex-duet-github`；同时将根目录的 `LICENSE` 复制到该安装目录以保留许可证：

- 个人使用：`~/.agents/skills/codex-duet-github/`，Windows 对应用户目录下的 `.agents\skills\codex-duet-github\`。
- 指定仓库使用：`<目标仓库>/.agents/skills/codex-duet-github/`。

安装后应直接存在 `codex-duet-github/SKILL.md`、`codex-duet-github/agents/openai.yaml` 和 `codex-duet-github/LICENSE`，不要多嵌套一层 `skill/`。根目录 README 无需安装。

可从已经发布的任务分支或合并后的默认分支下载文件；未合并时不要假设默认分支已有完整 Skill。若同名安装已经存在，先检查内容，再决定更新，不直接覆盖。安装副本的后续更新需要同步复制。

Codex 会自动发现变化；若没有显示，重启 Codex。在 Skill 选择器中选中它，或输入 `$codex-duet-github` 明确调用。仅把源项目放在普通代码目录下不等于安装。本项目不自动修改个人 Codex 配置。

## 前置条件

- Codex 已连接当前环境提供的 GitHub 插件，账户有目标仓库读写权限。
- 插件能读取默认分支/完整 SHA、创建分支、发布 Commit 并读取验证结果；“连接成功”不保证具备所有写能力。
- 本地完整目标项目可读取、可写入，所需构建/测试工具可执行。
- Codex 有可用的 Browser / Computer Use 通道，本次选定浏览器已登录网页版 ChatGPT。
- ChatGPT 自身已连接 GitHub，并获得目标仓库读取权限。这与 Codex 的 GitHub 连接是两项独立检查。

只检查实际使用的路径。例如选定内置浏览器时，不要求 Chrome 同时登录。首次运行将通过真实消息和已知 Commit 文件核对 ChatGPT 的仓库访问。

插件无法创建新仓库时，需要用户先创建并初始化目标仓库、授权插件，然后继续。404 也可能意味着未授权，不一定是不存在。所有 GitHub 仓库操作都走 Codex GitHub 插件，不自动用本地 Git、gh 或网页编辑替代。

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

Review 检查 `BASE_SHA..最新 REVIEW_SHA` 的累计修改。只选 Review 不会自动要求 Plan；只选 Discussion 仍由 Codex 自行规划、实施、验证与发布，并实际讨论关键方案。重要分歧无法达成共识时请用户决策。成功后询问是否合并回原分支并删除任务分支。

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

## 许可证

本项目采用 [Apache License 2.0](LICENSE)（SPDX：`Apache-2.0`）。
