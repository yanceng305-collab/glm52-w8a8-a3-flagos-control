# 组件、910C 与 GLM-5.2-W8A8 兼容矩阵

调查日期：2026-08-24

## 组件版本矩阵

| 组件 | 官方推荐/实际CI | 客户合规候选 | A3/910C专用 | GLM-5.2-W8A8必需 | 状态/边界 |
|---|---|---|---:|---:|---|
| Host OS/CANN | Host CANN版本不参与container tuple；只保留Host runtime条件 | 不bind-mount Host Toolkit | 是 | Driver/runtime层必需 | Boundary Confirmed；Host CANN差异不再分析 |
| Container base/OS | Docs：`v0.24.0rc1-a3`；artifact未建立 | Exact digest `sha256:1c36469f...` provisional carrier | 是 | 必需 | Official `releases/v0.24.0rc` A3 nightly；不是rc1 release image；实际OS/runtime preflight |
| Architecture | aarch64 | 按现场aarch64复核 | 是 | 必需 | Confirmed job metadata |
| Python | rc1 source recipe曾使用3.12 | actual carrier preflight | 否 | 必需 | 不从release recipe预设provisional artifact |
| Driver | Host mounted | 产品/CANN兼容版本 | 是 | 必需 | exact Unknown |
| Firmware | Host side | 与Driver/CANN匹配 | 是 | 必需 | exact Unknown |
| Container CANN | rc1 source recipe曾使用9.0.1 | actual carrier preflight；Host CANN irrelevant | 是 | 必需 | Provisional artifact exact version Unknown before preflight |
| Container ATB/NNAL | rc1 source recipe有对应组件 | actual carrier inventory冻结 | 是 | 部分native op必需 | Exact packages Unknown before preflight |
| HCCL | torch-npu/CANN内 | matching v0.24 stack | 是 | A3至少2 devices | v0.2.1 TP2 evidence；new-main A3未验证 |
| torch | rc1 source recipe曾使用2.10.0 | actual carrier preflight | 否 | 必需 | Provisional artifact不预设 |
| torch-npu | official evidence有多个2.10 family row | actual carrier preflight | 是 | 必需 | Target exact Unknown before preflight |
| vLLM | FL contract固定`v0.24.0` | actual carrier preflight必须验证0.24 compatibility；0.20.2仅maintenance/reference | 否 | 必需 | 不由nightly名称预设embedded identity |
| vLLM native ext | empty build无device ext | 无 | 否 | 非必需 | Confirmed |
| vllm-plugin-FL | observed main`a9435a34...`/tree`e5e073ed...`；v0.2.1=`92a6f767...` | Primary new main；mutation时重冻结 | 否 | 必需 | Branch migration Confirmed；main moving |
| `VLLM_VENDOR` | unset | unset | Ascend语义 | 必需配置 | Confirmed；`ascend`无效 |
| FlagGems | README tag`v5.3.4@f7c55cb2...` | preferred非mandatory；exact tag | 否 | per-op可选 | Confirmed source pin；new-main A3 E2E Unknown |
| triton/compiler provider | rc1 source recipe曾使用triton-ascend 3.2.1 | actual carrier inventory后执行FlagTree replacement | 是 | Triton路径需一个provider | 初始distribution/version Unknown；final需single coherent provider |
| FlagTree | FL README`0.6.1rc1+ascend3.5@9a90fddf...` | A2 final provider；替换actual conflicting provider | wheel是 | Triton路径需要 | Runtime transaction gate；single coherent provider或STOP |
| FlagCX | CI absent；README ref v0.13.0 | baseline absent | adaptor-specific | baseline非必需 | Optional；910C E2E Unknown |
| msModelSlim | master`e1009c9`；GLM5.2 feature`f8e5bed` | runtime先不装 | 配方含A3 tag | 重新量化时producer必需 | 工具侧Confirmed；公开A3 log Unknown |
| Quant format | CI无W8A8 | AscendV1 vs compressed-tensors待ADR | 模型/栈特定 | 必需 | Interface Missing/Conflicting |
| compressed-tensors | rc1 source constraint曾为`>=0.11.0` | actual resolved version + artifact ADR | 否 | 选CT路线时必需 | Provisional artifact exact Unknown |
| transformers | rc1 source recipe曾使用5.13.0 | actual carrier preflight | 否 | 必需 | Provisional artifact exact Unknown；FL compatibility待验 |
| NumPy | v0.24 requirements未exact pin | actual carrier inventory | 否 | 依赖 | Exact Unknown；不沿用old CI pin |
| Mooncake | rc1 source recipe曾使用`0.3.11.post1` | actual carrier preflight；baseline nonessential | 否 | 非必需 | Provisional artifact exact Unknown；optional for A2 |
| Ray | >=2.47.1,<=2.48.0 | local-MP baseline absent | 否 | 单机TP非必需 | Optional |
| FL Ascend custom ext | 无；Python-only | 无 | 是 | 非必需 | Confirmed |
| vllm-ascend image/distribution | v0.24.0rc1 A3 official documented tag；artifact unavailable | Exact digest release-branch nightly允许作A2 provisional carrier；FL-only smoke可在disposable container卸载 | 是 | Carrier candidate；FlagOS ownership必需 | Digest Confirmed；内部distribution/version/import由preflight |

v0.24 tuple与冲突证据见[`OFFICIAL-V024-BASELINE-RESEARCH.md`](OFFICIAL-V024-BASELINE-RESEARCH.md)；历史neutral tuple见[`R0-CONTAINER-TUPLE-RESOLUTION.md`](R0-CONTAINER-TUPLE-RESOLUTION.md)。

## 910C 成熟度

| 功能域 | 状态 | 边界 |
|---|---|---|
| FL platform/NPU init | Confirmed | 公开CI多日成功 |
| Qwen3.6-27B dense TP2 inference/serve | Confirmed on v0.2.1-era stack | 单节点2 logical devices；new v0.24 canary待重冻 |
| Qwen3.6-35B-A3B BF16 MoE TP2 | Confirmed on v0.2.1-era stack | 不能自动外推new main/provider tuple |
| Benchmark | Confirmed smoke only | 非性能baseline |
| Unit/functional | Confirmed with exclusion | `ops/test_ops_correctness.py`排除 |
| HCCL TP2 | Confirmed | 单节点 |
| Exact 910C SoC dispatch | Missing | FL无910B/910C guard |
| A2 environment identity/preparation | **Ready / previous attempt STOP before container** | exact digest、Git bundle、actual inventory、provider/FlagGems/FL闭环 |
| A2 shared-NPU tiny smoke | Ready after environment PASS | 同一任务使用共享NPU 12+13；仅tiny torch_npu与Dispatch op |
| Official coexistence runtime provenance | Partially Confirmed / dynamic audit deferred on-demand | static ownership与CI platform activation有证据；完整operator/import/native/compiler trace在Eager Correctness后按需触发，不阻塞Baseline Benchmark |
| FlagTree profile | Ready experiment / runtime Unknown | A2卸载carrier compiler后安装rc1并审计ownership |
| Dense MLA | Missing | placeholder |
| Sparse MLA/DSA/SFA | Missing | 显式NotImplemented |
| GLM-5 on 910C | Unknown/Missing backend | README非交叉矩阵 |
| GLM-5.2-Slim init/weight load | Confirmed PARTIAL on NVIDIA only | TP16/2-node NCCL；不证明Ascend/W8A8 |
| GLM-5.2-W8A8 on 910C | Missing / Unknown E2E | new main有模型contract，Ascend mandatory closure未闭合 |
| Generic vLLM0.24 Indexer framework | Implemented | `deepseek_v2.Indexer` + upstream `SparseAttnIndexer`；native platform list不含Ascend |
| Ascend/910C reachable Indexer closure | Missing / Unwired | backend、kernel owner、sparse MLA/DSA路径未闭合 |
| GLM-5.2 Ascend E2E Indexer | Missing | 无目标CI/E2E |
| Compressed-tensors W8A8 contract | Upstream vLLM0.24/CT present；current FL custom validator absent | v0.2.1 validator仅maintenance reference；target contract需重审 |
| FL W8A8 loading/packed glue | **Absent from current main** | v0.2.1 `w8a8/packed.py`存在但new main tree仅剩`quant_linear.py`；replacement path Unknown |
| OOT/NPU INT8 Linear candidate | Missing | upstream CPU/CUDA/ROCm candidates不支持Ascend OOT/NPU |
| Ascend 910C W8A8 Linear runtime | Missing | contract/glue有，NPU kernel与E2E未闭合 |
| W8A8 MoE | Implemented but unverified | 仅unit/mock |
| AscendV1 runtime in FL | Missing | FL无reader；carrier中的vllm-ascend reader可作contract reference，不能自动视为FL owner |
| MTP | vLLM0.24 structure implemented/unverified | 无target 910C E2E；first eager仍后置MTP |
| EP/DP | Implemented but unverified | 无公开910C case |
| Multi-node HCCL | Unknown | 无目标run |
| FlagCX | Implemented but unverified | CI未启用 |
| True graph | Open development/unverified | case名不足；平台关闭compile/graph |
| Profile/multistream | Open development/unverified | 不得前置 |

官方证据：[run 32287718197](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197)、[inference](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197/job/96182249510)、[serving](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197/job/96182249383)。

## GLM-5.2-W8A8 × FlagOS × 910C

| 能力 | Owner/实现 | 状态 | 结论 |
|---|---|---|---|
| `glm_moe_dsa` config | FL new-main bridge + transformers | Implemented but unverified on target | bridge不等于5.2-W8A8/910C闭环 |
| Model class | vLLM0.24 `GlmMoeDsaForCausalLM` / DeepSeekV2-derived | **Implemented** | NVIDIA partial已完成构造/weight loading |
| IndexShare owner/shared | vLLM0.24 model contract | Implemented but target unverified | 从“0.20 baseline Missing”升级为0.24 contract audit |
| Shared top-k reuse | vLLM0.24 sparse path | Implemented but target unverified | 需真实checkpoint/owner microgate |
| Shared checkpoint ownership | vLLM0.24 + target loader | Implemented contract / unverified | 需manifest key audit |
| MTP iteration reuse | vLLM0.24 | Implemented structure / unverified | first eager仍关闭MTP |
| Dense MLA | FL placeholder | **Missing** | 构造报错 |
| Sparse MLA/DSA/SFA | FL selector | **Missing** | 显式拒绝 |
| Generic Indexer model/framework | vLLM0.24 `deepseek_v2.Indexer` + `SparseAttnIndexer` + FL schemas | **Implemented** | 不等于Ascend native/backend可达 |
| Ascend/910C Indexer backend/kernel closure | Generic framework + target backend/kernel owner | **Missing / Unwired** | Ascend backend、kernel owner、metadata/cache与sparse MLA/DSA路径未闭合 |
| GLM-5.2 Ascend E2E Indexer | GLM sparse MLA/DSA完整链 | **Missing** | 当前910C CI无GLM/Indexer case |
| Indexer cache kernels | vLLM/FlagGems静态code | Implemented but unverified | 不能绕过上述可达性缺口 |
| MTP model construction | vLLM DeepSeekMTP | Implemented but unverified | 不绕开MLA/Indexer |
| Router sigmoid/top-k | vLLM+FL | Implemented but unverified for GLM | Qwen不能证明GLM组合 |
| BF16 routed experts | FL Ascend adapter | Implemented but unverified for GLM | Qwen只证明primitive |
| Shared expert | vLLM+FL | Implemented but unverified | 受Linear/quant约束 |
| ModelSlim W8A8 generation | msModelSlim GLM5.2 adapter | Confirmed tool-side | A3 verified metadata；公开log Unknown |
| Attention linear quant | msModelSlim | Confirmed tool-side | `wk/weights_proj`等排除 |
| Indexer `wq_b` | msModelSlim | Confirmed tool-side | pattern覆盖 |
| Indexer `wk/weights_proj` | msModelSlim | Confirmed excluded | 保留非W8A8 |
| Routed/shared experts quant | msModelSlim | Confirmed tool-side | dynamic W8A8；router排除 |
| Router quant | msModelSlim | Confirmed excluded | float路径 |
| MTP quant coverage | msModelSlim adapter | Implemented but unverified per-module | 无逐模块公开精度 |
| IndexShare adapter | msModelSlim | Implemented but unverified E2E | full/shared + MTP full |
| `concat_and_cache_mla` | FL schema + FlagGems generic Triton implementation | Implemented pieces / **Unwired for Ascend FL** | `bb439d`真实failure；FlagGems可达性与910C correctness Unknown |
| `concat_and_cache_mla_rope_fused` | FL schema | **Missing / Unwired** | current source未找到Ascend backend |
| AscendV1 reader | 当前仅在vllm-ascend找到 | **Missing in FL / contract pending** | 是否需要FL实现由真实artifact决定 |
| Compressed-tensors W8A8 config/checkpoint contract | upstream vLLM0.24 + compressed_tensors | Implemented upstream / target unverified | FL v0.2.1 validator已不在new main，必须从实际artifact重新审计 |
| FL W8A8 packed loading/glue | v0.2.1 `CompressedTensorsPackedW8A8Int8` | **Maintenance reference only / absent current main** | 不能假设old glue仍进入0.24 runtime |
| OOT/NPU INT8 Linear kernel candidate | vLLM0.24 selector + FL/FlagGems/vendor paths | **Unknown / re-audit required** | 0.20.2 Missing结论不能直接移植到0.24；仍无910C PASS |
| Ascend 910C W8A8 Linear runtime | contract + loading + NPU kernel + E2E | **Missing / unverified closure** | 必须按FlagGems/vendor/reference重新gap confirmation |
| CT W8A8 MoE | FL Triton adapter | Implemented but unverified | compiler/layout/scale待验 |
| Eager full load | 全栈 | Unknown | 多前置Missing |
| Single-node capacity-valid | Hardware+runtime | Unknown | aggregate只说明可能 |
| TP/DP/EP | vLLM/FL/HCCL | Unknown for model | 不静态冻结 |
| Two-node HCCL | torch-npu/HCCL | Unknown | 无目标run |
| Two-node FlagCX | FL+FlagCX | Implemented but unverified | 收益/稳定性Unknown |
| Graph/MTP/multistream | 全栈 | Unknown | eager后置 |

## Minimal Eager Execution Closure

### Dense MLA fallback判定

| 候选 | 官方/代码状态 | Correctness判定 | Closure结论 |
|---|---|---|---|
| Transformers eager attention | eager kernel存在，但`GlmMoeDsaAttention`仍执行Indexer top-k并生成sparse mask | 保留DSA语义 | 不是Dense fallback；DSA/Indexer仍必须 |
| vLLM 0.24 primary GLM路径 | `index_topk`触发Indexer并设置`is_sparse=True` | 保留官方语义；NVIDIA partial进一步证明实际构造/weight load | MLA + DSA + Indexer必须 |
| `VLLM_MLA_DISABLE=1` | 关闭MLA optimization并选`DeepseekV2Attention`；GLM仍分配top-k buffer，而该class断言buffer必须为None | 代码路径不闭合；即使patch绕过也会删除sparse mask | **不是合法fallback** |
| 删除/覆盖`index_topk/indexer_types` | 官方config/model code无此支持模式 | 改变目标模型结构语义，无correctness证据 | **禁止用于首次验收** |

证据：官方[GLM-5.2 config](https://huggingface.co/zai-org/GLM-5.2/blob/b4734de4facf877f85769a911abafc5283eab3d9/config.json#L20-L26)、Transformers [DSA/Indexer eager路径](https://github.com/huggingface/transformers/blob/e0e7504bca2bfd1b85bb0eedb148f7b250226f06/src/transformers/models/glm_moe_dsa/modeling_glm_moe_dsa.py#L347-L470)、vLLM 0.23 [sparse MLA/Indexer构造](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py#L999-L1074)与[`VLLM_MLA_DISABLE`非闭合路径](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py#L1115-L1130)。

### 首次正确eager token的mandatory closure

| 能力 | 状态 | Microgate最小验收 |
|---|---|---|
| 真实GLM-5.2-W8A8 checkpoint/runtime contract | Mandatory | manifest、quant metadata、tensor/scale/packing被正确识别 |
| W8A8 Linear | Mandatory；Ascend runtime Missing | OOT/NPU kernel可达，小shape对reference正确，可支撑目标linear forward |
| W8A8 MoE | Mandatory；910C unverified | routed/shared experts、scale和router保留精度正确 |
| MLA | Mandatory；Ascend Missing | backend可达，最小prefill/decode正确 |
| DSA semantics / SFA或等价backend | Mandatory；Ascend Missing | sparse top-k attention与官方eager/reference一致 |
| Indexer full/shared | Mandatory；framework Implemented，Ascend closure Missing/Unwired | full层计算、shared层复用、cache/logits/top-k kernel owner明确 |
| IndexShare weight ownership | Mandatory；vLLM0.24 contract存在、target unverified | 真实manifest owner/shared keys加载无错配 |
| 必要MoE/router/shared-expert/TP-HCCL runtime | Mandatory according to first layout | 目标forward所需链路correctness通过 |
| eager | Mandatory | 关闭非必需特性后输出第一个正确token |
| BF16 full-model bring-up | Not acceptable substitute | 仅operator/reference/debug microtest |
| MTP/graph/multistream/FlagCX/fusion | Not required | 后续阶段 |

结论：第一次目标模型eager closure必须是`GLM-5.2-W8A8 + MLA + DSA/SFA + Indexer + W8A8 Linear/MoE + required runtime + eager`。当前没有证据支持把DSA/Indexer后移。

模型证据：[GLM-5.2要求vLLM0.23+](https://huggingface.co/zai-org/GLM-5.2/blob/b4734de4facf877f85769a911abafc5283eab3d9/README.md)、[IndexShare/MTP config](https://huggingface.co/zai-org/GLM-5.2/blob/b4734de4facf877f85769a911abafc5283eab3d9/config.json)、[vLLM0.24实现](https://github.com/vllm-project/vllm/blob/ee0da84ab9e04ac7610e28580af62c365e898389/vllm/model_executor/models/deepseek_v2.py)、[FL GLM partial commit](https://github.com/flagos-ai/vllm-plugin-FL/commit/bb439d028479475a965712e08ce0b955fe02aafb)。

## 容量/并行

- **Confirmed physical specification：** 华为官方 Atlas 800I A3 为 `8 × 128GB = 1024GB`，[见官方技术规格](https://e.huawei.com/cn/products/computing/ascend/atlas-800i-a3)。
- **User-confirmed runtime presentation：** 当前Host边界冻结为`16 × 64GB logical devices = 1024GB aggregate`；后续container/device trace仍需验证全部16个device可见和单device可用HBM，不能把aggregate直接当KV/workspace余量。
- **Parameter-count source conflict：** 当前[vLLM官方GLM-5.2 recipe](https://recipes.vllm.ai/zai-org/GLM-5.2)写约743B total / 39B active；其他model-hub metadata可能显示约753B。计划采用“来源冲突”标记，不用任一摘要数字替代真实checkpoint manifest。
- 第三方W8A8 artifact大小只能作为非权威旁证，不能冻结full-model capacity或静态余量。
- Full-model capacity必须重新读取本项目真实GLM-5.2-W8A8 checkpoint manifest，逐项计入packed weights、scales、保留float tensor、workspace、KV、communication/allocator headroom，并结合Stage A实际logical-device topology做per-rank placement。
- Canary仍用TP2；full model不预先冻结TP/DP/EP或logical-device数量。
- 第二台不是当前前置；只由Capacity失败或明确scale-out/communication Stage触发。

Old Indexer/W8A8 links above are v0.2.1-era maintenance evidence, not new-main ownership. Primary evidence now includes[vLLM0.24 GLM/Indexer model](https://github.com/vllm-project/vllm/blob/ee0da84ab9e04ac7610e28580af62c365e898389/vllm/model_executor/models/deepseek_v2.py)、[upstream SparseAttnIndexer platform limit](https://github.com/vllm-project/vllm/blob/ee0da84ab9e04ac7610e28580af62c365e898389/vllm/model_executor/layers/sparse_attn_indexer.py#L474-L488)、[FL current-main quantization tree](https://github.com/flagos-ai/vllm-plugin-FL/tree/a9435a34dcd7d0a38e3a853535947371a6c62205/vllm_fl/quantization)和[`bb439d` GLM partial](https://github.com/flagos-ai/vllm-plugin-FL/commit/bb439d028479475a965712e08ce0b955fe02aafb)。
