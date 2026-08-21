# 技术决策记录

更新时间：2026-08-21

| ID | 决策 | 状态 | 理由 / 证据 | 重审条件 |
|---|---|---|---|---|
| D-001 | 新代码基线只从 `flagos-ai/vllm-plugin-FL` 操作时 current `main` HEAD 开始；legacy只作 history | Proposed for approval | 用户硬约束；当前调查 HEAD `38e7dbc` | 用户批准 repo transition 后冻结新 SHA |
| D-002 | 官方 A3 CI stack 只作 reference oracle，不作客户 formal environment | Evidence-backed proposed | image含完整 vllm-ascend package/custom kernels，违反正式环境约束 | 官方发布 clean-room CI 或客户改变约束 |
| D-003 | 第一 clean-room 候选为 `R0-clean = CI tuple - vllm-ascend`，从零构建且从未安装 vllm-ascend | Inferred / requires experiment | 最大限度保持单变量，同时满足 package/runtime 禁令 | 910C negative audit/canary失败 |
| D-004 | `R0-clean` compiler profile 先使用 actual CI 的 `triton-ascend==3.2.1`；FlagTree 作为独立 `R1-compiler` candidate | Proposed | README FlagTree 与 CI/FlagGems pins冲突，不能叠装或合成版本 | 官方冻结新一致 tuple 或独立实验完成 |
| D-005 | baseline communication 使用 HCCL；FlagCX 后置为独立变量 | Evidence-backed proposed | current 910C CI 走 HCCL；FlagCX未安装/未E2E | 客户明确强制FlagCX或独立验证通过 |
| D-006 | 首个严格官方 910C-backed canary 使用 Qwen3.6-27B TP2 eager | Evidence-backed proposed | current CI matrix 中最小真实成功模型 | 官方出现更小的同栈910C E2E |
| D-007 | GLM vLLM 版本不在本轮猜测冻结；canary后设 Contract Gate | Required | FL 0.20.2 与 GLM-5.2 vLLM>=0.23硬冲突 | Contract Gate ADR完成 |
| D-008 | AscendV1 与 compressed-tensors 的 quant contract 不在本轮猜测选择 | Required | ModelSlim能产出不等于FL能读；真实checkpoint未知 | manifest/layout审计和spike完成 |
| D-009 | “FlagOS原生”暂按 package/runtime independence 定义：允许官方FL维护的adapter与torch-npu/CANN下游 | Awaiting customer confirmation | 符合用户明确禁止项；当前FL含历史adapted源码 | 客户若也禁止历史来源，则项目需判定当前官方main不合规 |
| D-010 | 第二台A3不是 research、Clean Provenance 或 canary 前置 | Proposed | 单机足以完成这些目标；完整模型aggregate容量理论可能 | Capacity gate或scale-out目标触发 |
| D-011 | repository mutation 必须在 owner/preflight/影响报告获确认后执行 | Required | 保护branches/tags/PR #1；rename-alone不释放fork slot | 用户明确确认操作序列 |

## 明确拒绝的路线

- vllm-ascend image 作为 formal base；
- 安装/运行 vllm-ascend package 后再卸载；
- force-push 官方 main 覆盖 legacy；
- 删除、detach/recreate legacy；
- 从 `ascend-model-migration` 或旧控制面继续新项目；
- 把 README “GLM-5 Supported” 与 “Ascend Supported” 做笛卡尔积；
- 把 ModelSlim A3 verified tag当成 FL runtime E2E；
- 在 eager correctness/baseline 前进入性能优化。

## 待形成 ADR 的决策

### ADR-P01：GLM-5.2 vLLM 语义路线

候选：vLLM 0.23 最小 uplift；vLLM 0.24/dev + Ascend integration；0.20.2 精准 backport。比较 API/worker/model-runner 差异、IndexShare/MTP语义、910C canary回归、上游维护成本后决定。

### ADR-P02：W8A8 artifact contract

候选：在 FL 原生实现 AscendV1 loader/runtime；或把真实 artifact 转为 compressed-tensors 且逐模块证明语义等价。必须覆盖 attention linear、Indexer、routed/shared experts、router exclusion、MTP 和 scale layout。

### ADR-P03：Compiler profile

R0-clean 用 triton-ascend 3.2.1 复刻实际CI；R1 单独测试 FlagTree。不得同环境叠装；记录 `triton` distribution owner、backend、最小kernel、FlagGems与W8A8 MoE结果。
