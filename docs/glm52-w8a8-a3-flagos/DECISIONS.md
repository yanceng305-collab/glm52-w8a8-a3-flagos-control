# 技术决策记录

更新时间：2026-08-21

| ID | 决策 | 状态 | 理由 / 证据 | 重审条件 |
|---|---|---|---|---|
| D-001 | 新代码基线只从`flagos-ai/vllm-plugin-FL`建仓时current `main`开始；legacy只作history | Implemented | 2026-08-21冻结official commit`92a6f767...`/tree`e610bc58...`并精确复制 | control批准新的upstream同步 |
| D-002 | 官方 A3 CI stack 只作 reference oracle，不作客户 formal environment | **Superseded by D-044～D-049** | 该结论错误地把carrier中存在vllm-ascend package等同于runtime违规；compatibility数据保留 | 不再作为当前决策 |
| D-003 | 第一 clean-room 候选为 `R0-clean = CI tuple - vllm-ascend`，从零构建且从未安装 vllm-ascend | **Superseded by D-044～D-049** | “package必须缺席”前提已撤销；neutral tuple只保留为reference candidate | 不再作为当前决策 |
| D-004 | `R0-clean` compiler profile 先使用 actual CI 的 `triton-ascend==3.2.1`；FlagTree 作为独立 `R1-compiler` candidate | **Partially superseded by D-047** | 版本冲突证据仍有效，但compiler owner必须由runtime provider trace决定，不能由预选neutral tuple冻结 | provider trace或official tuple变化 |
| D-005 | baseline communication 使用 HCCL；FlagCX 后置为独立变量 | Evidence-backed proposed | current 910C CI 走 HCCL；FlagCX未安装/未E2E | 客户明确强制FlagCX或独立验证通过 |
| D-006 | 首个严格官方 910C-backed canary 使用 Qwen3.6-27B TP2 eager | Evidence-backed proposed | current CI matrix 中最小真实成功模型 | 官方出现更小的同栈910C E2E |
| D-007 | GLM vLLM 版本不在本轮猜测冻结；canary后设 Contract Gate | Required | FL 0.20.2 与 GLM-5.2 vLLM>=0.23硬冲突 | Contract Gate ADR完成 |
| D-008 | AscendV1 与 compressed-tensors 的runtime artifact contract不在本轮猜测选择 | Required | FL compressed-tensors W8A8 contract和packed loading/glue已Implemented；但AscendV1 reader、OOT/NPU INT8 Linear kernel与真实checkpoint格式仍未闭合 | manifest/layout审计和spike完成 |
| D-009 | “FlagOS原生”按package/runtime/environment independence定义 | **Superseded by D-044** | package/image存在性不能证明实际backend ownership；历史adapted来源与动态runtime调用必须分开判断 | 不再作为当前决策 |
| D-010 | 第二台A3不是research、Runtime Provenance或canary前置 | Proposed | Host边界已确认16×64GB logical devices/1024GB aggregate；真实checkpoint footprint、container可用量和KV/workspace余量仍Unknown | Capacity gate或scale-out目标触发 |
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
| D-021 | vLLM-Ascend允许作为Ascend/910C技术reference，生产实现必须spec-first进入`FlagGems/vendor.ascend/Reference -> torch_npu/CANN`的FlagOS ownership链 | Required；boundary refined by D-044～D-049 | 需要其hardware contract与primitive经验；carrier/package存在允许，但新增实现不得绕过FlagOS ownership绑定vllm-ascend backend | 客户扩大源码研究限制、trace发现实际参与或上游许可变化 |
| D-022 | 禁止通过改名、换目录或机械重构隐藏vLLM-Ascend源码来源 | Required | 技术参考不等于源码复制；实际复用/派生必须满足license与attribution | 对应上游license或法律审查变化 |
| D-023 | 参考优先级为FlagOS跨平台实现 → official vLLM/Transformers → vLLM-Ascend硬件参考 | Required | 先保持FlagOS架构一致性和模型contract，再吸收910C硬件知识 | official ownership结构变化 |
| D-024 | 正式环境要求vllm-ascend image/distribution/module/entry point全部不存在，且从未安装 | **Superseded by D-044～D-049** | 该presence-based门禁与official Ascend Dockerfile实际路线不一致；不删除历史记录 | 不再作为当前决策 |
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
| D-035 | R0-clean tuple改由Container官方证据和candidate image静态解析 | **Superseded by D-044～D-049** | Host CANN不再是tuple输入仍由D-038保留；neutral tuple不再是正式路线 | 不再作为当前决策 |
| D-036 | Host/现有Python/Docker中存在vllm-ascend只记录为现场“污染”事实；正式container仍须从未安装 | **Superseded by D-044** | 安装存在性现在只记录为环境inventory，不使用“污染”作合规结论 | runtime provenance trace |
| D-037 | 本机已有neutral candidate时仍不在research轮启动container | Satisfied for current round | 已记录CANN901 candidate；本轮只做OCI/official source研究 | 用户批准后续container实验 |
| D-038 | R0软件tuple以Container为边界；Host CANN版本不参与选择 | Required | 容器化部署只依赖Host Driver/Firmware/runtime/device/network/disk/model条件；原则上不bind-mount Host Toolkit | 未来方案显式挂载Host Toolkit |
| D-039 | R0-primary采用`cann:9.0.1-a3-ubuntu22.04-py3.12`完整Container tuple | **Superseded as formal route by D-045** | OCI/兼容性研究保留；neutral base不再是唯一合法R0 | 不再作为默认执行路线 |
| D-040 | R0-primary继续使用official vLLM0.20.2 empty与FL`92a6f767` | **Partially superseded by D-045** | vLLM/FL接口基线证据保留；其来源应优先遵循official carrier并由trace验证 | GLM Contract Gate或official baseline变化 |
| D-041 | 条件fallback为CANN900/Python311 CI-equivalent-minus-vllm-ascend tuple | **Superseded as formal fallback by D-045** | tuple兼容性数据保留，不再作为预先批准fallback | runtime trace/故障证据要求新决策 |
| D-042 | CANN8.5.0不是R0 fallback | Required | Host已有版本不构成Container fallback理由 | 新的Container级official证据和control决策 |
| D-043 | Candidate tag必须在执行前核对local RepoDigest并做container内vllm-ascend package/module absence audit | **Partially superseded by D-046** | digest固定仍Required；package absence不再是acceptance，改为inventory + runtime participation trace | provenance trace PASS |
| D-044 | “FlagOS原生/客户路线”按**实际模型执行ownership**判定，不按vllm-ascend image/package存在性判定 | Required | official Dockerfile使用vllm-ascend A3 carrier并通过`VLLM_PLUGINS=fl`激活FL；FL静态链路进入`PlatformFL/WorkerFL/ModelRunnerFL/Dispatch` | 客户书面扩大边界或runtime trace反证 |
| D-045 | 下一候选环境优先遵循official FL Ascend Dockerfile carrier路线；neutral CANN R0-P1/R0-F1降为reference evidence | Evidence-backed proposed / experiment pending | official current main `92a6f767`明确`FROM quay.io/ascend/vllm-ascend:v0.20.2rc1-a3`并设置FL selectors | exact digest、现场兼容或trace失败 |
| D-046 | vllm-ascend distribution/module/entry point的installed/discoverable状态是inventory事实，不是单独的PASS/FAIL门禁 | Required | CI保留package但激活FL；只有动态import/call与backend ownership能证明是否参与执行 | trace发现实际参与 |
| D-047 | Triton类路径必须追踪为`FL/FlagGems kernel -> Triton API -> FlagTree或triton-ascend provider -> CANN`；非Triton路径可直接`PyTorch/torch_npu -> CANN` | Required / provider Unknown | FL源码调用Triton API或torch_npu；实际provider不能由README/安装名猜测 | runtime provider trace |
| D-048 | `vendor.ascend`是`vllm_fl` ownership下的backend，不是vllm-ascend runtime wrapper；adapted source provenance与runtime dependency分开记录 | Static-checked；runtime verification pending | class path位于`vllm_fl.dispatch.backends.vendor.ascend`，attention直接调用torch_npu；部分文件保留Adapted from声明 | runtime module/call trace反证或客户扩大源码边界 |
| D-049 | Runtime Provenance Trace是910C Canary前的新门禁；若发现任何`vllm_ascend`实际参与，只对具体调用做客户边界/替换判断 | Proposed / Not Ready | 静态代码能证明预期ownership，不能证明目标进程动态闭包 | 用户批准执行并完成trace |

## 明确拒绝的路线

- 仅因image/package名含`vllm-ascend`就判定不合规，或仅因FL静态树无direct import就宣称运行时独立；
- 未经trace与control审查，让`vllm_ascend` backend绕过FlagOS Dispatch拥有目标模型关键执行；
- 为制造“clean”表象而先安装/使用再卸载，且不保留来源与运行时证据；
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

先在official carrier中记录`triton` distribution owner、active driver/provider及最小kernel下游，确认实际是FlagTree还是triton-ascend；不得由README或package名猜测。只有runtime provenance后，才决定是否需要独立compiler对照profile；FlagTree是provider，不是vllm-ascend backend代理。

### ADR-P04：Minimal Eager Execution Closure

当前结论为“无合法Dense MLA fallback”。任何候选fallback必须同时满足：官方config/model code可达、不删除`index_topk/indexer_types`、与Transformers eager DSA输出在批准tolerance内一致、目标vLLM与910C backend有证据。在此之前，DSA/SFA与Indexer不得从第一次W8A8 eager closure后移。

### ADR-P05：Mandatory Capability Closure与vLLM-Ascend Reference

已知mandatory Missing/Unwired能力必须先经过gap contract，再在FlagOS ownership链中最小实现，最后通过对应microgate；不能以“已有参考代码”为理由跳过验收。Codex可研究vLLM-Ascend的行为和hardware contract；DeepSeek未来只能按implementation contract重新实现。若实际复用/派生源码，PR必须明确范围、许可证和attribution。详细任务规则见`tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md`。

补充路径规则：gap contract必须先审查FlagGems、vendor.ascend和NPU-resident Reference；存在任一合法correctness路径时不得仅因缺少FlagGems而进入实现。Reference路径的性能问题只在Eager Correctness后由profiling触发。
