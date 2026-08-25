# GLM-5.2-W8A8 × FlagOS × Ascend A3/910C

本仓库是项目控制面与长期事实源。它帮助新的 User、ChatGPT、Codex1、Codex2 或 DeepSeek 会话快速回答：项目为何存在、当前正式路线是什么、做到哪一门禁、Evidence 在哪里，以及下一步能否开始。

## Project Goal

目标是在 Ascend A3 / 910C 上，以 vLLM 0.24 和 vllm-plugin-FL 为框架入口，使 GLM-5.2-W8A8 的模型执行真实落入 FlagOS runtime/Dispatch ownership，完成可审计的模型加载、正确性、端到端推理与服务；正确性基线稳定后，再通过 benchmark、profiling 和单变量实验做性能优化。

## Customer Constraint

客户要求正式模型执行走 FlagOS 体系，而不是把 vllm-ascend 作为正式运行 backend。项目因此以 `PlatformFL → WorkerFL → ModelRunnerFL → FlagOS Dispatch` 的实际激活和逐算子 backend ownership 判定合规，合法下游包括 FlagGems、`vendor.ascend` 和 NPU-resident Reference。

Control repo 只把这一点记录为客户硬约束，没有记录客户选择 FlagOS 的商业或组织背景；新会话不得自行补猜理由。

vllm-ascend 仍可用于 Ascend/910C contract、primitive 和 correctness 行为的官方技术参考，也可出现在环境 carrier 中；package 存在不自动等于运行依赖。新增正式实现不得绕过 FlagOS ownership 直接绑定 vllm-ascend backend，发现动态调用时必须单独归因和决策。详见 [DECISIONS.md](docs/glm52-w8a8-a3-flagos/DECISIONS.md) 与 [FLAGOS-RUNTIME-MAP.md](docs/glm52-w8a8-a3-flagos/FLAGOS-RUNTIME-MAP.md)。

## Target

| 项目 | 当前正式目标 |
|---|---|
| Model | GLM-5.2-W8A8；真实 checkpoint，W8A8 是首次模型正确性硬门禁 |
| Hardware | Ascend A3 / 910C；已知项目边界为 16×64 GB logical devices，真实模型 placement 仍待 checkpoint/runtime 证据 |
| Framework | vLLM 0.24.0 + vllm-plugin-FL frozen v0.24 project baseline |
| FlagOS components | PlatformFL、WorkerFL、ModelRunnerFL、Dispatch；FlagGems / `vendor.ascend` / NPU-resident Reference |
| Ascend stack | FlagTree/Triton 或 PyTorch/torch_npu → CANN → A3/910C；HCCL 为默认通信 backend，FlagCX 后置 |

正式 Code baseline 是 `a9435a34dcd7d0a38e3a853535947371a6c62205`（tree `e5e073edf4b65c053e954d78d20365aab0e1f46b`），primary integration branch 是 `project/glm52-w8a8-v024`。这是项目冻结快照，不等于随时间前进的 upstream HEAD。Code repo 的 `main@92a6f767...` 仅为 v0.2.1 / vLLM 0.20.2 maintenance/reference，不是当前开发线。

## Runtime Architecture

```text
vLLM 0.24
  ↓ VLLM_PLUGINS=fl
PlatformFL → WorkerFL → ModelRunnerFL
  ↓ CachedOp / FlagOS Dispatch
FlagGems | vllm_fl vendor.ascend | NPU-resident Reference
  ↓                                      ↓
Triton API → FlagTree              PyTorch / torch_npu
  └───────────────────→ CANN ←──────────┘
                         ↓
                    Ascend A3 / 910C
```

冻结源码已静态确认到 Platform/Worker/ModelRunner/Dispatch 的链路。Original + supplement + final verification联合审查已确认tiny torch_npu、FlagTree kernel、FlagGems direct和FL/Dispatch四条路径显式在NPU执行、backend正确且无silent CPU fallback；A2 scope-limited Acceptance为`ACCEPTED`。它仍未运行模型、服务、collective、benchmark或profile。

## Repositories

| 仓库 | Current role |
|---|---|
| [Control repo](https://github.com/yanceng305-collab/glm52-w8a8-a3-flagos-control)（本仓库） | PLAN、STATUS、DECISIONS、tasks、research、Evidence pointer、immutable results、Codex1 Acceptance 与仓库治理 |
| [Code repo](https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos) | 实际 vllm-plugin-FL 代码、task branches、commits、tests 与 PR；standalone repo，official upstream 通过 remote/记录关联 |
| [Official FL upstream](https://github.com/flagos-ai/vllm-plugin-FL) | 查当前官方实现；只有经 Control 决策冻结的 exact commit 才成为项目 baseline |
| [Legacy fork](https://github.com/yanceng305-collab/vllm-plugin-FL) | 历史 A2 branches/tags/PR #1；保留不变，不是正式 Code repo，也不得作为新 task 起点 |

Code 修改遵循 `project integration branch → task branch → edit/test → commit/push → PR → Codex1 review`。不需要修改 FL 的 task 可写 `Code PR = N/A`。详细仓库身份与迁移记录见 [CODE-REPOSITORY-BASELINE.md](docs/glm52-w8a8-a3-flagos/CODE-REPOSITORY-BASELINE.md) 和 [CODE-REPOSITORY-MIGRATION-V024.md](docs/glm52-w8a8-a3-flagos/CODE-REPOSITORY-MIGRATION-V024.md)。

## Current Phase

**A2 clean-room environment/runtime/compiler/Dispatch tiny gate已ACCEPTED。** 下一Stage已选择为910C Canary；Task `CANARY-V024-QWEN36-27B-TP2`已Draft，但仍是**Waiting User input / Not Ready**，未执行、未修改Code。

| 字段 | 当前记录 |
|---|---|
| Task | `A2-V024-CLEANROOM-CANN900-PY311` |
| Run | `20260824T080753Z` |
| Codex2 execution | **PASS reported** |
| Control | original sync `31d20df...`；supplement `cdb586af...`；final pointer `6f369d8a...`；snapshot record `031a2db...` |
| Codex1 Acceptance | **ACCEPTED（A2 scope-limited）** |
| FL change / PR | 无 FL 修改；`N/A` |
| Final verification | `20260825T030520Z` PASS / **ACCEPTED**；Control `6f369d8a...` |
| Default runtime | `flagos-glm52-a3-runtime:v024-cann900-py311` / image ID `sha256:e1a89dca...`（host-local） |
| Next Stage | 910C Canary：`Qwen/Qwen3.6-27B@cea40373...` complete BF16，TP2 eager offline |
| Canary status | **Waiting User input / Not Ready**；exact model path、two-device resources与dispatch待确认 |

Accepted tuple：Python 3.11.15、CANN 9.0.0、torch 2.10.0+cpu、torch_npu 2.10.0.post2、vLLM 0.24.0、FlagTree 0.6.1+ascend3.5 / Triton 3.5.1、FlagGems 5.3.4、FL `a9435a34...`。未发现错误base/runtime、old-runtime reuse、mixed provider、CPU fallback、NPU失败或FL未记录修改。

阶段边界：baseline research、Code repo v0.24 migration与A2 clean-room gate已完成；旧v0.24 Python3.12 A2和copied-runtime Python3.11只保留历史/feasibility价值。Canary contract已完成，但User必须手动提供/确认官方ModelScope完整权重，Codex1/Codex2不得下载或silent substitute。

## What Is NOT Done

Tiny runtime/operator/Dispatch smoke PASS **不等于**“GLM-5.2-W8A8 已可运行”。当前仍未完成：

- `CANARY-V024-QWEN36-27B-TP2`的model construction、real weight load、TP2 eager generation与Codex1 Acceptance；
- 真实 GLM-5.2-W8A8 checkpoint manifest、quantization/layout contract 与容量/placement 闭合；
- 目标模型级 MLA、DSA/SFA、Indexer、MLA KV/cache ops、W8A8 Linear/MoE 的 Ascend 可达性、correctness 与缺口闭合；
- 完整 WorkerFL / ModelRunnerFL 模型路径、目标模型构造、权重加载、首个正确 eager token；
- GLM-5.2-W8A8 离线 E2E inference、serve、目标多卡/collective 和稳定性验收；
- baseline benchmark、profiling、性能优化、组合优化与按需 scale-out。

A2已不再阻塞推进，但上述模型级工作仍必须分别经过control-approved canary、GLM contract/capability/capacity、first eager/eager correctness、benchmark/profile/optimize等独立合同与验收。本Acceptance不降低任何后续标准。

## Where to Find Information

| 问题 | 正式入口 |
|---|---|
| 当前状态与门禁 | [STATUS.md](docs/glm52-w8a8-a3-flagos/STATUS.md) |
| 技术路线与 supersede 关系 | [DECISIONS.md](docs/glm52-w8a8-a3-flagos/DECISIONS.md) |
| 阶段路线 | [PLAN.md](docs/glm52-w8a8-a3-flagos/PLAN.md)；其中动态状态须与 STATUS/result index 交叉核对 |
| Code baseline / integration branch | [CODE-REPOSITORY-BASELINE.md](docs/glm52-w8a8-a3-flagos/CODE-REPOSITORY-BASELINE.md) |
| v0.24 branch migration | [CODE-REPOSITORY-MIGRATION-V024.md](docs/glm52-w8a8-a3-flagos/CODE-REPOSITORY-MIGRATION-V024.md) |
| Runtime ownership / architecture | [FLAGOS-RUNTIME-MAP.md](docs/glm52-w8a8-a3-flagos/FLAGOS-RUNTIME-MAP.md) |
| Compatibility 与未闭合能力 | [COMPATIBILITY-MATRIX.md](docs/glm52-w8a8-a3-flagos/COMPATIBILITY-MATRIX.md) |
| Environment / official baseline research | [ENVIRONMENT-RESEARCH.md](docs/glm52-w8a8-a3-flagos/ENVIRONMENT-RESEARCH.md)、[OFFICIAL-V024-BASELINE-RESEARCH.md](docs/glm52-w8a8-a3-flagos/OFFICIAL-V024-BASELINE-RESEARCH.md) |
| Clean-room environment reconstruction | [A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md](docs/glm52-w8a8-a3-flagos/A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md) |
| Current 910C Canary contract | [CANARY-V024-QWEN36-27B-TP2.md](docs/glm52-w8a8-a3-flagos/tasks/CANARY-V024-QWEN36-27B-TP2.md) |
| Canary Codex2 prompt | [CODEX2-CANARY-V024-QWEN36-27B-TP2-PROMPT.md](docs/glm52-w8a8-a3-flagos/tasks/CODEX2-CANARY-V024-QWEN36-27B-TP2-PROMPT.md) |
| 首次 eager 的能力边界 | [MINIMAL-EAGER-EXECUTION-CLOSURE.md](docs/glm52-w8a8-a3-flagos/MINIMAL-EAGER-EXECUTION-CLOSURE.md) |
| Task contracts / prompts | [`tasks/`](docs/glm52-w8a8-a3-flagos/tasks/) |
| Result 索引 / Acceptance | [results/INDEX.md](docs/glm52-w8a8-a3-flagos/results/INDEX.md) |
| 最新 immutable result | [20260824T080753Z.md](docs/glm52-w8a8-a3-flagos/results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md) |
| Evidence 与仓库治理 | [REPOSITORY-AND-EVIDENCE-RULES.md](docs/glm52-w8a8-a3-flagos/REPOSITORY-AND-EVIDENCE-RULES.md) |

旧 v0.2.1/vLLM 0.20.2 A2、neutral-base-only / copied-runtime 路线、旧 carrier/Docker/prompt 和 legacy fork 均保留作 Historical / Reference；带 `Superseded`、`Paused`、`Not Ready` 的资料不得作为当前授权。冲突时遵循 [AGENTS.md](AGENTS.md#动态信息与冲突处理) 的优先级。

## Server Workspace

正式长期目录由仓库治理文件冻结：

```text
/data/tiankuan/zyg/
├─ repos/       # Control 与 Code 的唯一长期 Git working trees
├─ evidence/    # <task-id>/<run-id>/；run 关闭后不可修改
├─ artifacts/   # wheel、bundle、source archive 等复用物及 SHA256
├─ work/        # <task-id>/<run-id>/；可删除 build/staging/临时解包
└─ legacy/      # 历史路径索引；不得作为正式源码
```

服务器代理切换时优先继续已有 container、run-id、Evidence、work、Code branch 和安装状态；只有环境损坏或 task 明确要求 clean-room 才重建。

后续另行授权任务默认优先复用[`A2 clean-room reconstruction`](docs/glm52-w8a8-a3-flagos/A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md)记录的local runtime snapshot。Launch前必须核对image ID/runtime tuple、task-specific device与Docker参数，并验证`/data` bind下formal FL checkout的branch/HEAD/tree/clean状态；snapshot不是registry-pinned、portable或model-ready artifact。

## Upstream References

Control repo 保存“本项目已冻结和已验证的事实”；公开、动态的上游状态按需回到官方来源核对：

- [vllm-plugin-FL](https://github.com/flagos-ai/vllm-plugin-FL)：核对 PlatformFL / WorkerFL / ModelRunnerFL、Dispatch、pyproject、Dockerfile 与 CI 的当前实现。
- [FlagGems](https://github.com/flagos-ai/FlagGems)：核对 Dispatch backend、Ascend backend、operator 与 compiler provider 支持。
- [FlagTree](https://github.com/flagos-ai/FlagTree)：核对 Triton/Ascend compiler provider、分支、构建与版本事实。
- [vLLM](https://github.com/vllm-project/vllm)：核对目标版本接口、依赖、GLM 模型、cache 与 quantization contract。
- [vLLM-Ascend](https://github.com/vllm-project/vllm-ascend)：Ascend/vLLM hardware contract 与兼容性参考；不代表当前正式 FlagOS runtime backend。
- [Ascend PyTorch / torch_npu](https://github.com/Ascend/pytorch)：核对 torch_npu 版本兼容、安装与 NPU API。
- [CANN 9.0.x 文档](https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/900/index/index.html) 与 [CANN container images](https://github.com/Ascend/cann-container-image)：核对 CANN 配套、安装、release notes、容器构建与 tag/Dockerfile。
- [GLM-5.2 model](https://huggingface.co/zai-org/GLM-5.2) 与 [Transformers](https://github.com/huggingface/transformers)：核对目标模型 config、checkpoint contract 与参考语义。

只有 exact commit/version 已成为 Decision、frozen baseline、formal dependency 或真实实验 provenance 时，才把它具体记录进 Control repo。
