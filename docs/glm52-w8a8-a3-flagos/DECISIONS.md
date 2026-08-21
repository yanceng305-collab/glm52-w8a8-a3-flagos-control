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
| D-008 | AscendV1 与 compressed-tensors 的runtime artifact contract不在本轮猜测选择 | Required | FL compressed-tensors W8A8 contract和packed loading/glue已Implemented；但AscendV1 reader、OOT/NPU INT8 Linear kernel与真实checkpoint格式仍未闭合 | manifest/layout审计和spike完成 |
| D-009 | “FlagOS原生”当前按package/runtime/environment independence定义；允许研究vLLM-Ascend并允许official FL维护的adapter与torch-npu/CANN下游 | Working boundary | 客户明确禁止runtime/package/环境依赖，尚未禁止开发阶段研究或official FL历史adapted来源 | 仅当客户以后明确扩大到历史adapted来源时重审 |
| D-010 | 第二台A3不是 research、Clean Provenance 或 canary 前置 | Proposed | 华为只直接确认单机物理规格8×128GB；runtime logical-device呈现、真实checkpoint footprint和可用余量均Unknown，必须在Stage A/Capacity gate实测重算 | Capacity gate或scale-out目标触发 |
| D-011 | repository mutation 必须在 owner/preflight/影响报告获确认后执行 | Required | 保护branches/tags/PR #1；rename-alone不释放fork slot | 用户明确确认操作序列 |
| D-012 | Indexer状态按framework、Ascend可达闭环、GLM-5.2 E2E三层记录 | Evidence refinement | Generic FL Indexer framework已Implemented；Missing仅指Ascend/910C backend/kernel closure与GLM E2E | official main或目标E2E证据变化 |
| D-013 | W8A8 Linear按contract、packed glue、OOT/NPU kernel、910C runtime四层记录 | Evidence refinement | 前两层Implemented；OOT/NPU candidate与910C runtime Missing | 新NPU kernel或E2E证据出现 |
| D-014 | Full-model capacity不使用摘要参数量或第三方artifact冻结 | Required | 华为物理HBM为8×128GB；logical topology Unknown；vLLM recipe约743B与其他metadata约753B有来源冲突 | 真实checkpoint manifest与Stage A topology冻结 |
| D-015 | W8A8是第一次目标模型eager correctness硬门禁 | Required | 目标固定为GLM-5.2-W8A8；BF16只允许operator/reference/debug microtest，不能替代full-model bring-up | 用户改变目标checkpoint（当前不允许） |
| D-016 | Minimal Eager Execution Closure必须包含MLA + DSA/SFA + Indexer | Evidence-backed proposed | 固定证据集内：官方config固定index_topk/indexer_types；Transformers eager仍生成sparse mask；vLLM0.23没有合法Dense MLA fallback | 官方新增correctness-preserving fallback及目标平台证据 |
| D-017 | Capability Microgates只证明首次eager mandatory能力最小可用 | Required | 验收限于backend可达、小规模correctness、checkpoint/runtime contract、目标forward接口和NPU device/backend trace | Eager Correctness通过后进入更完整阶段 |
| D-018 | 首次eager前采用`gap confirmation -> Minimal Capability Implementation -> corresponding Microgate PASS` | Required | MLA/DSA/Indexer/W8A8等mandatory能力已静态确认存在Missing/Unwired，不能只检查后直接进入First Eager | 全部mandatory microgate PASS |
| D-019 | Minimal Capability Implementation只接收FlagGems/vendor.ascend/Reference三路均不可用后确认Missing/Unwired的首次eager mandatory能力 | Required | 防止把“没有FlagGems”误判成Missing，也防止范围扩张和提前优化；First Eager后的Minimal Compatibility只处理新暴露故障 | 新证据改变mandatory closure |
| D-020 | Mandatory capability原则上独立任务、独立branch、独立Draft PR | Required | 保持正确性归因、许可证/provenance和回滚边界 | 只有不可分割接口经Codex书面批准可合并 |
| D-021 | vLLM-Ascend允许作为Ascend/910C技术reference，生产实现必须spec-first进入`FlagGems/vendor.ascend/Reference -> torch_npu/CANN`的FlagOS ownership链 | Required | 需要其hardware contract与primitive经验，但正式环境和runtime依赖仍被禁止 | 客户扩大源码研究限制或上游许可变化 |
| D-022 | 禁止通过改名、换目录或机械重构隐藏vLLM-Ascend源码来源 | Required | 技术参考不等于源码复制；实际复用/派生必须满足license与attribution | 对应上游license或法律审查变化 |
| D-023 | 参考优先级为FlagOS跨平台实现 → official vLLM/Transformers → vLLM-Ascend硬件参考 | Required | 先保持FlagOS架构一致性和模型contract，再吸收910C硬件知识 | official ownership结构变化 |
| D-024 | 正式环境继续要求vllm-ascend image/distribution/module/entry point/runtime dependency全部不存在，且禁止先安装再卸载 | Required | 参考策略不改变客户运行环境约束 | 客户明确变更正式环境要求 |
| D-025 | FlagScale不作为首次GLM-5.2-W8A8 bring-up前置 | Required | 当前目标是直接闭合vLLM/FL/Worker/ModelRunner/Dispatch到910C的eager correctness；控制/编排集成后置 | 模型链稳定且进入Advanced Composition |
| D-026 | FlagGems为preferred而非mandatory实现来源 | Required | Bring-up验收关注correctness和FlagOS Dispatch可达性，不关注统一算子来源 | 性能阶段profile证明需替换现有路径 |
| D-027 | Gap Confirmation路径顺序为FlagGems → vendor.ascend → Reference/PyTorch；三者均不可用才实现新能力 | Required | 复用当前FlagOS合法路径，最小化首次eager前开发 | 某路径违反runtime边界或correctness失败 |
| D-028 | Reference路径必须NPU-resident并提供device/backend trace | Required | PyTorch tensor在NPU上可由torch_npu/CANN执行；静默CPU fallback不合法 | 目标backend提供等价、可审计的更直接证明 |
| D-029 | Reference性能优化后置到Eager Correctness后的profiling | Required | 首次bring-up不为FlagGems覆盖率或性能提前开发；瓶颈需测量证明 | Profile确认Reference为主要瓶颈 |

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

### ADR-P04：Minimal Eager Execution Closure

当前结论为“无合法Dense MLA fallback”。任何候选fallback必须同时满足：官方config/model code可达、不删除`index_topk/indexer_types`、与Transformers eager DSA输出在批准tolerance内一致、目标vLLM与910C backend有证据。在此之前，DSA/SFA与Indexer不得从第一次W8A8 eager closure后移。

### ADR-P05：Mandatory Capability Closure与vLLM-Ascend Reference

已知mandatory Missing/Unwired能力必须先经过gap contract，再在FlagOS ownership链中最小实现，最后通过对应microgate；不能以“已有参考代码”为理由跳过验收。Codex可研究vLLM-Ascend的行为和hardware contract；DeepSeek未来只能按implementation contract重新实现。若实际复用/派生源码，PR必须明确范围、许可证和attribution。详细任务规则见`tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md`。

补充路径规则：gap contract必须先审查FlagGems、vendor.ascend和NPU-resident Reference；存在任一合法correctness路径时不得仅因缺少FlagGems而进入实现。Reference路径的性能问题只在Eager Correctness后由profiling触发。
