# 组件、910C 与 GLM-5.2-W8A8 兼容矩阵

调查日期：2026-08-21

## 组件版本矩阵

| 组件 | 官方推荐/实际CI | 客户合规候选 | A3/910C专用 | GLM-5.2-W8A8必需 | 状态/边界 |
|---|---|---|---:|---:|---|
| Host OS | 完整Unknown；kernel `5.10.0-216.0.0.115.oe2203sp4.aarch64` | 现场后冻结 | 是 | 基础必需 | metadata Confirmed；完整OS Unknown |
| Container OS | Ubuntu 22.04 | Ubuntu 22.04 | 否 | 必需 | Confirmed |
| Architecture | aarch64 | 按现场aarch64复核 | 是 | 必需 | Confirmed job metadata |
| Python | 3.11.15 | 3.11.15 | 否 | 必需 | Confirmed |
| Driver | Host mounted | 产品/CANN兼容版本 | 是 | 必需 | exact Unknown |
| Firmware | Host side | 与Driver/CANN匹配 | 是 | 必需 | exact Unknown |
| CANN | 9.0.0 A3 | 9.0.0 A3 | 是 | 必需 | Confirmed；patch/build Unknown |
| ATB/NNAL | image中有`latest` path | CANN base配套 | 是 | 部分native op可能必需 | exact/necessity Unknown |
| HCCL | torch-npu/CANN内 | matching stack | 是 | TP>1必需 | TP2 Confirmed；独立版本Unknown |
| torch | 2.10.0 | 2.10.0 first | 否 | 必需 | image recipe Confirmed |
| torch-npu | 2.10.0；另有2.10.0.post2 | 2.10.0 first；post2另测 | 是 | 必需 | CI Confirmed；suffix Conflicting |
| vLLM | `v0.20.2@bc150f5`, empty | Canary同版；GLM待Contract Gate | 否 | 必需 | Canary Confirmed；GLM需>=0.23 |
| vLLM native ext | empty build无device ext | 无 | 否 | 非必需 | Confirmed |
| vllm-plugin-FL | `v0.2.1-rc0@38e7dbc` | 操作时current main重冻结 | 否 | 必需 | Confirmed snapshot |
| `VLLM_VENDOR` | unset | unset | Ascend语义 | 必需配置 | Confirmed；`ascend`无效 |
| FlagGems | `3e6528cf`, metadata5.0.2 | R0同pin | 否 | 当前FL链必需 | Confirmed CI；README pin不同 |
| triton-ascend | 3.2.1 | R0同版 | 是 | FlagGems/Triton kernels必需 | Confirmed CI |
| FlagTree | CI absent；README0.4；其他metadata指0.5/3.5 | R0 absent；R1待测 | wheel是 | 是否必需Unknown | **Conflicting** |
| FlagCX | CI absent；README ref v0.13.0 | baseline absent | adaptor-specific | baseline非必需 | Optional；910C E2E Unknown |
| msModelSlim | master`e1009c9`；GLM5.2 feature`f8e5bed` | runtime先不装 | 配方含A3 tag | 重新量化时producer必需 | 工具侧Confirmed；公开A3 log Unknown |
| Quant format | CI无W8A8 | AscendV1 vs compressed-tensors待ADR | 模型/栈特定 | 必需 | Interface Missing/Conflicting |
| compressed-tensors | image requirements>=0.11.0；FL用其contract | 随批准vLLM冻结 | 否 | 选CT路线时必需 | exact需重算 |
| transformers | CI image5.5.3；GLM config写5.12.0 | 随vLLM/模型contract冻结 | 否 | 必需 | Conflicting |
| NumPy | 1.26.4 | 1.26.4 | 否 | 依赖 | Confirmed |
| Mooncake | v0.3.8.post1 | baseline absent | 否 | 非必需 | Optional |
| Ray | >=2.47.1,<=2.48.0 | local-MP baseline absent | 否 | 单机TP非必需 | Optional |
| FL Ascend custom ext | 无；Python-only | 无 | 是 | 非必需 | Confirmed |
| vllm-ascend | package/ext installed | **禁止且必须不存在** | 是 | 客户路线不允许 | CI presence Confirmed；clean independence Unknown |

## 910C 成熟度

| 功能域 | 状态 | 边界 |
|---|---|---|
| FL platform/NPU init | Confirmed | 公开CI多日成功 |
| Qwen3.6-27B dense TP2 inference/serve | Confirmed | 单节点2 logical devices |
| Qwen3.6-35B-A3B BF16 MoE TP2 | Confirmed | 当前CI |
| Benchmark | Confirmed smoke only | 非性能baseline |
| Unit/functional | Confirmed with exclusion | `ops/test_ops_correctness.py`排除 |
| HCCL TP2 | Confirmed | 单节点 |
| Exact 910C SoC dispatch | Missing | FL无910B/910C guard |
| Clean-room无vllm-ascend | Unknown | 无negative control |
| FlagTree profile | Unknown | CI没装 |
| Dense MLA | Missing | placeholder |
| Sparse MLA/DSA/SFA | Missing | 显式NotImplemented |
| GLM-5 on 910C | Unknown/Missing backend | README非交叉矩阵 |
| GLM-5.2 on 910C | Missing | 版本语义+backend缺口 |
| Generic FL Indexer framework | Implemented | `SparseAttnIndexerFL/BF16`与DeepSeek-V4 glue存在 |
| Ascend/910C reachable Indexer closure | Missing / Unwired | backend、kernel owner、sparse MLA/DSA路径未闭合 |
| GLM-5.2 Ascend E2E Indexer | Missing | 无目标CI/E2E |
| Compressed-tensors W8A8 config/checkpoint contract | Implemented | canonical W8A8 subset validator存在 |
| FL W8A8 loading/packed glue | Implemented | packed参数与scheme patch存在 |
| OOT/NPU INT8 Linear candidate | Missing | upstream CPU/CUDA/ROCm candidates不支持Ascend OOT/NPU |
| Ascend 910C W8A8 Linear runtime | Missing | contract/glue有，NPU kernel与E2E未闭合 |
| W8A8 MoE | Implemented but unverified | 仅unit/mock |
| AscendV1 runtime | Missing | FL无reader |
| MTP | General implemented/unverified；5.2 semantics Missing on0.20.2 | 无910C E2E |
| EP/DP | Implemented but unverified | 无公开910C case |
| Multi-node HCCL | Unknown | 无目标run |
| FlagCX | Implemented but unverified | CI未启用 |
| True graph | Open development/unverified | case名不足；平台关闭compile/graph |
| Profile/multistream | Open development/unverified | 不得前置 |

官方证据：[run 32287718197](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197)、[inference](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197/job/96182249510)、[serving](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197/job/96182249383)。

## GLM-5.2-W8A8 × FlagOS × 910C

| 能力 | Owner/实现 | 状态 | 结论 |
|---|---|---|---|
| `glm_moe_dsa` config | FL bridge/transformers | Implemented but unverified | bridge不等于5.2语义完整 |
| Model class | vLLM DeepSeekV2-derived | Implemented but unverified | 0.20.2有类名 |
| IndexShare owner/shared | vLLM0.23+ | **Missing baseline** | 0.20.2每层建Indexer |
| Shared top-k reuse | vLLM0.23+ | **Missing** | correctness contract |
| Shared checkpoint ownership | vLLM0.23+ | **Missing** | 0.20.2期待不匹配 |
| MTP iteration reuse | vLLM0.23+ | **Missing** | 0.20.2无该语义 |
| Dense MLA | FL placeholder | **Missing** | 构造报错 |
| Sparse MLA/DSA/SFA | FL selector | **Missing** | 显式拒绝 |
| Generic FL Indexer framework | `SparseAttnIndexerFL/BF16` + DeepSeek-V4 attention glue | **Implemented** | custom-op/CachedOp与BF16调用点存在；不等于Ascend可达 |
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
| AscendV1 reader | 仅vllm-ascend | **Missing in FL** | 客户禁止依赖 |
| Compressed-tensors W8A8 config/checkpoint contract | FL validation | **Implemented** | canonical dynamic-token/per-channel INT8 subset存在 |
| FL W8A8 packed loading/glue | `CompressedTensorsPackedW8A8Int8` | **Implemented** | packed参数创建、unpack、scheme patch存在 |
| OOT/NPU INT8 Linear kernel candidate | vLLM selector + FL OOT list glue | **Missing** | 原始candidate仅CPU/CUDA/ROCm，platform gate不接受Ascend OOT/NPU |
| Ascend 910C W8A8 Linear runtime | contract + loading + NPU kernel + E2E | **Missing** | 前两层已实现，kernel/E2E未闭合 |
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
| vLLM 0.23正常GLM路径 | `index_topk`触发Indexer并设置`is_sparse=True` | 保留官方语义 | MLA + DSA + Indexer必须 |
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
| IndexShare weight ownership | Mandatory；current0.20.2 baseline Missing | 真实manifest owner/shared keys加载无错配 |
| 必要MoE/router/shared-expert/TP-HCCL runtime | Mandatory according to first layout | 目标forward所需链路correctness通过 |
| eager | Mandatory | 关闭非必需特性后输出第一个正确token |
| BF16 full-model bring-up | Not acceptable substitute | 仅operator/reference/debug microtest |
| MTP/graph/multistream/FlagCX/fusion | Not required | 后续阶段 |

结论：第一次目标模型eager closure必须是`GLM-5.2-W8A8 + MLA + DSA/SFA + Indexer + W8A8 Linear/MoE + required runtime + eager`。当前没有证据支持把DSA/Indexer后移。

模型证据：[GLM-5.2要求vLLM0.23+](https://huggingface.co/zai-org/GLM-5.2/blob/b4734de4facf877f85769a911abafc5283eab3d9/README.md)、[IndexShare/MTP config](https://huggingface.co/zai-org/GLM-5.2/blob/b4734de4facf877f85769a911abafc5283eab3d9/config.json)、[vLLM0.23实现](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py)。

## 容量/并行

- **Confirmed physical specification：** 华为官方 Atlas 800I A3 为 `8 × 128GB = 1024GB`，[见官方技术规格](https://e.huawei.com/cn/products/computing/ascend/atlas-800i-a3)。
- **Unknown runtime presentation：** “16 × 64GB logical devices”当前不得作为事实或容量分片前提；Stage A未来必须以真实`npu-smi`、`torch.npu.device_count()`和device properties冻结logical-device count、单device可用HBM与拓扑。
- **Parameter-count source conflict：** 当前[vLLM官方GLM-5.2 recipe](https://recipes.vllm.ai/zai-org/GLM-5.2)写约743B total / 39B active；其他model-hub metadata可能显示约753B。计划采用“来源冲突”标记，不用任一摘要数字替代真实checkpoint manifest。
- 第三方W8A8 artifact大小只能作为非权威旁证，不能冻结full-model capacity或静态余量。
- Full-model capacity必须重新读取本项目真实GLM-5.2-W8A8 checkpoint manifest，逐项计入packed weights、scales、保留float tensor、workspace、KV、communication/allocator headroom，并结合Stage A实际logical-device topology做per-rank placement。
- Canary仍用TP2；full model不预先冻结TP/DP/EP或logical-device数量。
- 第二台不是当前前置；只由Capacity失败或明确scale-out/communication Stage触发。

Indexer与W8A8分层的固定源码证据：[`SparseAttnIndexerFL`](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/ops/sparse_attn_indexer.py#L388-L460)、[`SparseAttnIndexerBF16`](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/ops/sparse_attn_indexer_bf16.py#L476-L538)、[compressed-tensors W8A8 validator](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/quantization/compressed_tensors.py#L83-L153)、[packed W8A8 glue](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/quantization/w8a8/packed.py#L40-L149)、[vLLM INT8 candidate table](https://github.com/vllm-project/vllm/blob/bc150f50299199599673614f80d12a196f377655/vllm/model_executor/kernels/linear/__init__.py#L151-L159)。
