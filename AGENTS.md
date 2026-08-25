# Agent Guide

## 项目一句话

本项目要在 Ascend A3 / 910C 上，通过 FlagOS 的 vLLM 插件、运行时 ownership 与 Dispatch 路径完成 GLM-5.2-W8A8 推理适配，先建立正确性，再做可归因的性能优化。

面向人的项目地图见 [README.md](README.md)。动态状态以 [STATUS.md](docs/glm52-w8a8-a3-flagos/STATUS.md) 和 [results/INDEX.md](docs/glm52-w8a8-a3-flagos/results/INDEX.md) 为准。

## 角色

- **User / Project Owner**：提供客户需求、服务器/硬件/模型边界；决定任务是否下发、资源与风险边界；批准重大路线、版本、CANN 主版本、baseline 与架构变化，拥有最终决策权。
- **ChatGPT**：与 User 做日常技术讨论、官方资料/source 核对、日志与代理结果独立分析、任务拆分和跨代理协调；可建议 PASS、REJECT 或 FOLLOW-UP，但不替代 Codex1 写正式 Acceptance。
- **Codex1**：本地规划与正式审查代理；维护 PLAN、DECISIONS、STATUS 和 task contract，独立审查 Code/Evidence/result/PR，并在 Control repo 写 `ACCEPTED / REJECTED / NEEDS-FOLLOWUP`。默认不在目标服务器安装、跑模型或代替现场调试。
- **Codex2**：服务器主执行代理；按已下发 task 调查、安装、编译、调试、修改代码、测试、采集 Evidence，生成 immutable result，并负责 Code task branch、commit、push、PR 与 Control result 同步。普通技术问题可自主处理；重大路线变化必须提出 `Decision requested`。
- **DeepSeek**：备用服务器执行代理，仅在 Codex2 不可用、额度/环境异常或 User 要求时接替；与 Codex2 共用同一个 **Server Execution Agent** 技术路线和既有现场状态。

```text
User / Project Owner
├─ ChatGPT：讨论、独立分析、跨代理协调
├─ Codex1：正式规划、Task、Review、Acceptance
└─ Codex2：服务器主执行
   └─ DeepSeek：备用服务器执行
```

User 的明确最终决定高于其他代理意见。ChatGPT 可独立复核，但 Control repo 的正式 Acceptance 由 Codex1 写入。

## 仓库边界

- **Control repo（本仓库）**：项目目标、PLAN、STATUS、DECISIONS、task/prompt、research、Evidence pointer、immutable result 与 Codex1 Acceptance。
- **Code repo**：[`yanceng305-collab/vllm-plugin-FL-a3-flagos`](https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos)，保存实际 FL 代码、task branch、commit 和 PR。正式 baseline、integration branch 与 legacy/upstream 关系见 [CODE-REPOSITORY-BASELINE.md](docs/glm52-w8a8-a3-flagos/CODE-REPOSITORY-BASELINE.md)。
- **Server Evidence**：真实日志、manifest、checksum 和 runtime facts；不代替长期 Git working tree，也不等同于 immutable result。

不得把 FL 源码改动放入 Control repo。

## 新会话读取顺序

### Codex1 / ChatGPT

1. `AGENTS.md`
2. `README.md`
3. `docs/glm52-w8a8-a3-flagos/STATUS.md`
4. `docs/glm52-w8a8-a3-flagos/DECISIONS.md`
5. `docs/glm52-w8a8-a3-flagos/results/INDEX.md`
6. 当前 task、对应 immutable result / Evidence pointer
7. 按 README 导航读取必要专题资料；只在需要时重新查官方 source

### Codex2 / DeepSeek

1. `AGENTS.md`
2. `README.md`
3. `docs/glm52-w8a8-a3-flagos/STATUS.md`
4. User 已下发的当前 task 合同及 prompt
5. 当前 run、result、Evidence、work/container 与安装状态
6. `CODE-REPOSITORY-BASELINE.md`、Code repo 当前 task branch/HEAD/tree/dirty state
7. `REPOSITORY-AND-EVIDENCE-RULES.md`

没有 User 下发的 ready task 时，服务器执行代理不得把计划或历史 prompt 当作执行授权。

## Prompt / Task Conventions

- Prompt / task contract只规定目标、执行与风险边界、验收条件，以及完成任务必需的User-confirmed事实。
- 不无必要冻结具体命令、命令顺序或普通排障步骤；Codex1定义结果与Acceptance边界，Codex2自主选择边界内的普通执行和排障方法。
- 只有正式边界、可重现性或验收本身依赖某条命令时，才冻结该命令，并说明原因。
- 已在本`AGENTS.md`定义的长期角色、仓库、Evidence、续接与禁止规则不重复写入prompt；prompt只引用本文件并补充本任务差异。
- 下发或执行前先同步并核对最新正式GitHub状态；GitHub正式记录优先于本地旧副本、旧prompt和聊天摘录。

## 工作规则

### Codex1

- 修改前同步 Control `main`，从正式资料恢复事实；不依赖聊天记忆。
- 设计可执行、可停止、可验收的 task；独立审查三指针、Evidence、Code diff/PR 与结果边界。
- 只在完成独立审查后更新 Acceptance；不得把服务器 `Execution PASS` 自动写成 `ACCEPTED`。
- 重大路线与 User 决定冲突时停止推进并请求 User 决策。

### Codex2 / DeepSeek

- 只执行 User 正式下发的 task，遵守资源、写入目录、停止条件与 Decision 边界。
- 优先恢复已有 container、run-id、work、Evidence、Code task branch 和安装/实验状态；代理切换本身不是 clean-room 重建理由。
- 只有环境损坏或 task 明确要求 clean-room 时才新建环境/run；避免重复下载、安装和已完成实验。
- 代码从 project integration branch 派生 `task/<task-id>-<short-name>`，测试后 commit/push，并以 integration branch 为 base 建 PR；无需修改 FL 时 `Code PR = N/A` 合法。
- 若要改变正式路线、目标版本、CANN 主版本、project baseline、客户边界或重大架构，写 `Decision requested`，不得自行推进。

## Task → Evidence → Result → Acceptance

```text
Codex1 task contract + User dispatch
  → Codex2 或 DeepSeek 服务器执行
  → Server Evidence（raw logs / manifest / checksums）
  → immutable result + results/INDEX.md Control sync
  → Codex1 独立审查 Code / Evidence / result / PR
  → ACCEPTED / REJECTED / NEEDS-FOLLOWUP
```

每个 run 必须保留 Code、Control、Evidence 三指针。Immutable result 首次 push 后不修改；Acceptance 只更新 `results/INDEX.md` 或 `STATUS.md`。完整规则见 [REPOSITORY-AND-EVIDENCE-RULES.md](docs/glm52-w8a8-a3-flagos/REPOSITORY-AND-EVIDENCE-RULES.md)。

## 动态信息与冲突处理

不要在本文件写死当前 task/run、Acceptance、服务器占用、container 状态、最新 upstream HEAD、临时 task branch 或随上游变化的软件矩阵；分别去 STATUS、results/INDEX、task/result/Evidence、Code repo 与官方 source 核对。

冲突时按以下顺序裁决：

1. User 当前明确决定与真实当前 Git/ref 状态；
2. `DECISIONS.md` 中仍有效且未 supersede 的决定；
3. immutable result 对该次 run 的执行事实；
4. `results/INDEX.md` 对 Control Sync 与 Codex Acceptance 的状态；
5. 更晚的正式记录与 `STATUS.md` 当前门禁；
6. PLAN、research、旧 task/prompt 与历史路线仅作其标注范围内的参考。

发现过时、冲突或 superseded 内容时保留历史，但不得机械复制为 current。

## 禁止事项与官方资料

- 不 force push、不修改 immutable result、不直接开发 Code `main` 或 baseline branch、不把 legacy fork 当正式 Code repo。
- 不因 tiny tensor/operator/Dispatch smoke PASS 宣称 GLM-5.2-W8A8 已加载、正确推理或性能达标。
- 不未经 Acceptance / User 决策解锁下一 Stage，不把换代理当作重做现场的理由。
- 不把公开上游 README/矩阵整段复制进 Control；项目专属且冻结的事实写入本仓库，动态上游事实保留官方链接并按需重查。

官方入口及各自查询用途见 [README.md 的 Upstream References](README.md#upstream-references)。
