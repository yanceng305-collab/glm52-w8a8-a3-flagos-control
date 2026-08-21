# 技术决策记录

更新时间：2026-08-21

| ID | 决策 | 状态 | 理由 / 证据 | 重审条件 |
|---|---|---|---|---|
| D-001 | 新代码基线只从`flagos-ai/vllm-plugin-FL`建仓时current `main`开始；legacy只作history | Implemented | 2026-08-21冻结official commit`92a6f767...`/tree`e610bc58...`并精确复制 | control批准新的upstream同步 |
| D-002 | 官方 A3 CI stack 只作 reference oracle，不作客户 formal environment | Evidence-backed proposed | image含完整 vllm-ascend package/custom kernels，违反正式环境约束 | 官方发布 clean-room CI 或客户改变约束 |
| D-003 | 第一 clean-room 候选为 `R0-clean = CI tuple - vllm-ascend`，从零构建且从未安装 vllm-ascend | Inferred / requires experiment | 最大限度保持单变量，同时满足 package/runtime 禁令 | 910C negative audit/canary失败 |
| D-004 | `R0-clean` compiler profile 先使用 actual CI 的 `triton-ascend==3.2.1`；FlagTree 作为独立 `R1-compiler` candidate | Proposed | README FlagTree 与 CI/FlagGems pins冲突，不能叠装或合成版本 | 官方冻结新一致 tuple 或独立实验完成 |
| D-005 | baseline communication 使用 HCCL；FlagCX 后置为独立变量 | Evidence-backed proposed | current 910C CI 走 HCCL；FlagCX未安装/未E2E | 客户明确强制FlagCX或独立验证通过 |
| D-006 | 首个严格官方 910C-backed canary 使用 Qwen3.6-27B TP2 eager | Evidence-backed proposed | current CI matrix 中最小真实成功模型 | 官方出现更小的同栈910C E2E |
| D-007 | GLM vLLM 版本不在本轮猜测冻结；canary后设 Contract Gate | Required | FL 0.20.2 与 GLM-5.2 vLLM>=0.23硬冲突 | Contract Gate ADR完成 |
| D-008 | AscendV1 与 compressed-tensors 的runtime artifact contract不在本轮猜测选择 | Required | FL compressed-tensors W8A8 contract和packed loading/glue已Implemented；但AscendV1 reader、OOT/NPU INT8 Linear kernel与真实checkpoint格式仍未闭合 | manifest/layout审计和spike完成 |
| D-009 | “FlagOS原生”当前按package/runtime/environment independence定义；允许研究vLLM-Ascend并允许official FL维护的adapter与torch-npu/CANN下游 | Working boundary | 客户明确禁止runtime/package/环境依赖，尚未禁止开发阶段研究或official FL历史adapted来源 | 仅当客户以后明确扩大到历史adapted来源时重审 |
| D-010 | 第二台A3不是research、Clean Provenance或canary前置 | Proposed | Host边界已确认16×64GB logical devices/1024GB aggregate；真实checkpoint footprint、container可用量和KV/workspace余量仍Unknown | Capacity gate或scale-out目标触发 |
| D-011 | repository mutation 必须在 owner/preflight/影响报告获确认后执行 | Required | 保护branches/tags/PR #1；rename-alone不释放fork slot | 用户明确确认操作序列 |
| D-012 | Indexer状态按framework、Ascend可达闭环、GLM-5.2 E2E三层记录 | Evidence refinement | Generic FL Indexer framework已Implemented；Missing仅指Ascend/910C backend/kernel closure与GLM E2E | official main或目标E2E证据变化 |
| D-013 | W8A8 Linear按contract、packed glue、OOT/NPU kernel、910C runtime四层记录 | Evidence refinement | 前两层Implemented；OOT/NPU candidate与910C runtime Missing | 新NPU kernel或E2E证据出现 |
| D-014 | Full-model capacity不使用摘要参数量或第三方artifact冻结 | Required | 物理8×128GB与logical16×64GB已确认；vLLM recipe约743B与其他metadata约753B有来源冲突，且runtime余量未知 | 真实checkpoint manifest与container device/memory trace冻结 |
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
| D-030 | 正式A3代码仓库使用personal standalone而非formal fork | Implemented | `yanceng305-collab/vllm-plugin-FL-a3-flagos`已创建；legacy零变更，代码baseline与official精确相等 | 只有上游贡献流程形成硬需求时重审formal fork |
| D-031 | 新代码仓库`main`只保存control-approved official frozen baseline，禁止直接开发 | Required | 防止实验/legacy历史污染；capability使用独立branch和Draft PR | control明确批准baseline同步 |
| D-032 | Upstream同步采用control-approved exact-SHA fast-forward policy | Required | Standalone无GitHub fork sync；每次记录old/new SHA/tree和兼容影响，不产生额外baseline commit | official历史非fast-forward或项目版本路线变化 |
| D-033 | A3-CP-A1不再是当前tuple决策前置 | Superseded / Not Ready | 用户已确认Container边界所需Host事实；本轮禁止服务器/DeepSeek操作 | 未来需要补充Host runtime证据并获新授权 |
| D-034 | 若未来重新执行A3-CP-A1，唯一允许写入仍为当前WORKDIR下全新Evidence目录 | Dormant control | raw logs/report/checksum需保存；目录外只读，不自行换目录或提权 | 新任务授权 |
| D-035 | R0-clean tuple改由Container官方证据和candidate image静态解析 | Superseded by D-038～D-043 | Host CANN不再是tuple输入；runtime acceptance仍需后续实验 | Primary/fallback实验结果 |
| D-036 | Host/现有Python/Docker中存在vllm-ascend只记录为现场污染事实 | Required | 当前/历史存在不等于未来从零构建的R0-clean违规；正式container仍须从未安装vllm-ascend | R0-clean negative audit结果 |
| D-037 | 本机已有neutral candidate时仍不在research轮启动container | Satisfied for current round | 已记录CANN901 candidate；本轮只做OCI/official source研究 | 用户批准后续container实验 |
| D-038 | R0软件tuple以Container为边界；Host CANN版本不参与选择 | Required | 容器化部署只依赖Host Driver/Firmware/runtime/device/network/disk/model条件；原则上不bind-mount Host Toolkit | 未来方案显式挂载Host Toolkit |
| D-039 | R0-primary采用`cann:9.0.1-a3-ubuntu22.04-py3.12`完整Container tuple | Static decision / experiment pending | 本机已有candidate；OCI确认CANN901/Python3.12.13/NNAL；official A3 reference配套torch2.10/post2/triton3.2.1 | clean negative audit或Qwen canary失败且归因到tuple |
| D-040 | R0-primary继续使用official vLLM0.20.2 empty与FL`92a6f767` | Required for first clean/Qwen gate | 保持当前FL接口/CI contract；不声称满足GLM-5.2最终vLLM语义 | GLM Contract Gate批准0.23+ uplift/backport |
| D-041 | 条件fallback为CANN900/Python311 CI-equivalent-minus-vllm-ascend tuple | Conditional only | 仅当primary失败且证据归因到CANN901/Python312/post2组合时启用；不与primary并行 | 对应failure evidence |
| D-042 | CANN8.5.0不是R0 fallback | Required | Host已有版本不构成Container fallback理由 | 新的Container级official证据和control决策 |
| D-043 | Candidate tag必须在执行前核对local RepoDigest并做container内vllm-ascend negative audit | Required | OCI history支持neutral判断但不能证明本机tag内容和runtime package absence | Digest/negative audit PASS |

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
