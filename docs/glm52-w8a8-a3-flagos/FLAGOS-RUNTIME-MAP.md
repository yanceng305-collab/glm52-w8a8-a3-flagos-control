# FlagOS on Ascend 910C Runtime Ownership Map

Primary static review：FL observed new `main@a9435a34dcd7d0a38e3a853535947371a6c62205` / tree`e5e073ed...` + official vLLM0.24 `ee0da84...`。`92a6f767...`/vLLM0.20.2现为official v0.2.1 maintenance/reference；早期链接保留历史语境。

Branch migration没有改变顶层ownership模型：current main仍注册`PlatformFL`、指定`WorkerFL`并构造`ModelRunnerFL`，operator仍进入FlagOS Dispatch。但0.20.2-era capability状态不能未经审查直接复制到0.24；W8A8 selector/API、GLM cache ops和compiler profile均须重新gap confirmation。

## 真实调用链

```text
Environment carrier candidate
  quay.io/ascend/vllm-ascend:v0.24.0rc1-a3
  (CANN / torch-npu / empty vLLM / compiler-provider / installed packages)
  + VLLM_PLUGINS=fl, VLLM_FL_PLATFORM=ascend
    ↓
vllm.LLM / vllm serve
  -> vLLM EngineArgs / EngineCore / Scheduler / Executor
  -> platform entry point fl=vllm_fl:register
  -> PlatformFL(device=npu, dist=hccl, worker=WorkerFL)
  -> WorkerFL (NPU/distributed init, Ascend patches)
  -> ModelRunnerFL
  -> vLLM model loader / ModelRegistry
  -> vLLM model class
  -> vLLM layers + FL OOT layers + CachedOp dispatch
       -> FlagGems or FL vendor.ascend Triton kernel
            -> Triton API -> FlagTree provider OR triton-ascend provider -> CANN/AICore
       -> FL vendor.ascend non-Triton implementation -> PyTorch/torch_npu -> CANN/AICore
       -> NPU-resident Reference -> PyTorch/torch_npu -> CANN/AICore
  -> HCCL default / FlagCX optional
  -> Ascend 910C
```

`vllm-ascend` distribution may be installed in the carrier, but it is deliberately not drawn as a runtime hop: static FL source does not show such a hop. Whether the official coexistence process imports or calls any `vllm_ascend` code is **Unknown until Post-Eager Runtime Provenance Audit**；A2的FL-only smoke不会回答该问题。

## 接管边界

- vLLM拥有API、serve、scheduler、EngineCore、Executor、model loader/registry与GLM模型主体。
- FL接管platform registration、Worker、ModelRunner、OOT layers与operator dispatch。
- `WorkerFL`/`ModelRunnerFL`是FL维护的vLLM fork，不是vllm-ascend类。
- Dense Attention glue属于`vllm_fl.dispatch.backends.vendor.ascend`，kernel owner是torch-npu/CANN；该backend不是vllm-ascend wrapper。
- FlagGems负责ATen替换与部分activation/router/operator；不拥有当前Ascend dense attention，也没接通sparse MLA。
- FlagTree或triton-ascend是Triton compiler/provider，不是model/attention/quant owner，也不是vllm-ascend backend代理。
- FlagCX是optional collective/KV transfer；默认是HCCL。

## Ownership matrix

| 模块 | Owner / 文件或类 | 下游 | FlagOS原生 | torch-npu/CANN | 历史vllm-ascend | 成熟度 |
|---|---|---|---|---|---|---|
| API/LLM/serve | vLLM entrypoints | EngineCore | 否 | 间接 | 否 | Confirmed CI |
| Scheduler/Engine/Executor | vLLM `v1/engine` | WorkerWrapper | 否 | 间接 | 否 | Confirmed CI |
| Platform registration | FL `register` / `PlatformFL` | vLLM config/platform | 是 | 探测间接 | OOT设计相关 | Confirmed |
| Worker | FL `WorkerFL` | ModelRunnerFL | FL-owned fork | 是 | 无direct import | Static current main；Qwen910C仅v0.2.1-era |
| ModelRunner | FL `ModelRunnerFL` | vLLM model | FL-owned fork | 是 | 无direct import | Static current main；Qwen910C仅v0.2.1-era |
| Loader/registry | vLLM | Model class | 否 | 间接 | 否 | Confirmed |
| GLM config bridge | FL `GlmMoeDsaConfig` | transformers/vLLM | 是 | 否 | 否 | Implemented, no GLM E2E |
| GLM model class | vLLM0.24 `GlmMoeDsaForCausalLM` | MLA/Indexer/MoE | 否 | 间接 | 否 | Implemented；NVIDIA init/weight load PARTIAL |
| OOT layers | FL `ops/custom_ops.py` | FL dispatch | 是 | 视op | 否 | Confirmed framework |
| CachedOp dispatch | FL `dispatch` + `ascend.yaml` | flagos/vendor/reference | 是 | 视backend | 否 | Confirmed framework |
| Environment carrier | vLLM-Ascend v0.24 official documented A3 candidate | CANN9.0.1/torch-npu/empty vLLM0.24/compiler/packages | 环境层，不判ownership | 是 | package存在 | Source/docs Confirmed；artifact identity Unknown |
| Dense attention glue | FL `vendor/ascend/impl/attention.py` | torch-npu ops | adapter是 | 强 | 明确adapted | v0.2.1 Qwen910C Confirmed；new main unverified |
| Dense attention kernel | torch-npu/CANN | CANN/AICore | 否 | 核心 | API历史相关 | v0.2.1 Qwen910C Confirmed；new main unverified |
| Dense MLA | 无usable owner；placeholder | — | 否 | — | 对照实现未迁入 | **Missing** |
| Sparse MLA/DSA/SFA | Ascend selector显式拒绝 | — | 否 | — | 历史实现存在于vllm-ascend | **Missing** |
| IndexShare logic | vLLM0.24 model contract | Indexer | 否 | 间接 | 否 | Implemented structure；target unverified |
| Generic Indexer framework | vLLM0.24 `deepseek_v2.Indexer` + upstream `SparseAttnIndexer`；FL cache/index schemas | model -> custom ops | 混合 | backend-dependent | 否 | **Implemented contract**；upstream native list不含Ascend |
| Ascend/910C reachable Indexer backend/kernel closure | Generic framework存在，但Ascend backend、kernel owner与sparse MLA/DSA调用链未闭合 | — | 未闭合 | 预期依赖 | 否 | **Missing / Unwired** |
| GLM-5.2 Ascend E2E Indexer | GLM sparse MLA/DSA -> Indexer完整链 | — | 未闭合 | 预期依赖 | 否 | **Missing**；无910C E2E |
| MLA cache concat schemas | FL `_C_ops_schemas.py` | backend implementation | contract是 | 预期依赖 | `bb439d`真实gap | schema Implemented；Ascend `concat_and_cache_mla*` **Missing/Unwired** |
| ATen replacement | FlagGems enable + blacklist | Triton/torch fallback | 是 | fallback依赖 | 否 | Worker confirmed；覆盖有限 |
| RMSNorm/RoPE | FL Ascend adapter | torch-npu | adapter是 | 强 | RoPE有历史来源 | Confirmed链路 |
| Router | vLLM + FL GateLinear/router patches | FlagGems/pure torch | 混合 | 间接 | 否 | BF16 MoE confirmed |
| BF16 experts | FL `impl/fused_moe.py` | NPU grouped matmul/MoE | adapter是 | 强 | 未标direct | Qwen MoE confirmed；分支Unknown |
| EP dispatch/combine | vLLM managers | HCCL/FlagCX | 否 | backend相关 | 否 | Implemented, unverified EP |
| Compressed-tensors W8A8 contract | upstream vLLM0.24 + compressed_tensors | INT8 schemes | 上游owner | 间接 | 否 | Implemented upstream；target artifact unverified |
| FL packed W8A8 loading/glue | v0.2.1 `CompressedTensorsPackedW8A8Int8` | old vLLM scheme | maintenance-only | 间接 | 否 | **Absent current main**；0.24 replacement Unknown |
| AscendV1 description | FL无人读取；reader当前仅在vllm-ascend找到 | — | 否 | — | carrier package中的实现可作contract reference | **Missing in FL**；是否需要迁入待ADR |
| OOT/NPU INT8 Linear kernel candidate | vLLM0.24 selector + FL/FlagGems/vendor paths | — | 待重审 | 预期依赖 | 否 | **Unknown on 0.24**；old Missing不能直接沿用 |
| Ascend 910C W8A8 Linear runtime | contract + packed loading + NPU kernel +910C执行 | — | 未闭合 | 预期依赖 | 否 | **Missing**；无910C E2E |
| W8A8 MoE | FL oracle + vLLM Triton experts | Triton provider | 混合 | 预期 | 否 | Implemented, unverified910C |
| TP communication | torch distributed/HCCL | HCCL | 否 | 强 | 否 | v0.2.1 TP2 Confirmed；v0.24 A3至少2 devices待验 |
| FlagCX collective | FL communicator + FlagCX | adaptor/HCCL | FlagOS组件 | adaptor依赖 | 否 | Optional, unverified910C |
| P/D KV transfer | FL `FlagCXConnector` | libflagcx P2P | FlagOS组件 | adaptor依赖 | 否 | Implemented, unverified |
| Triton compiler/provider | Carrier triton-ascend3.2.1或README FlagTree rc1 | CANN codegen | provider相关 | 是 | 非backend代理 | 两者共享namespace、互斥；replacement transaction Unknown |
| `vllm_ascend` dynamic participation | installed carrier distribution / entry point | Unknown until post-eager A/B audit | 不属于FlagOS ownership | 可能间接 | 核查对象 | **Unknown**；presence不等于call |
| A3/910C FL extension | 无；`VLLM_VENDOR`只支持cuda | 下游二进制栈 | 否 | 依赖下游 | 否 | Not designed |

## Attention

- Dense：`PlatformFL -> vendor.ascend -> torch_npu`；Qwen3.6 TP2 910C E2E仅在v0.2.1-era Confirmed，new main/provider tuple待验。
- MLA：Ascend返回构造即报错的placeholder；Missing。
- DSA/SFA：`use_mla && use_sparse`直接 `NotImplementedError`；Missing。
- FlagGems内有DSA相关kernel不等于plugin backend可达；selector/metadata/cache/backend必须闭合。

## Indexer 分层判定

- **Generic FL Indexer framework — Implemented：** FL固定树包含 [`SparseAttnIndexerFL`](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/ops/sparse_attn_indexer.py#L388-L460)、[`SparseAttnIndexerBF16`](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/ops/sparse_attn_indexer_bf16.py#L476-L538)及其custom-op/CachedOp框架；DeepSeek-V4 attention保留Indexer integration point，[见generic构造点](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/ops/deepseek_v4_attention.py#L981-L992)，BF16路径会直接构造`SparseAttnIndexerBF16`，[见BF16构造点](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/vllm_fl/ops/deepseek_v4_attention_bf16.py#L1175-L1186)。
- **Ascend/910C reachable backend/kernel closure — Missing / Unwired：** 当前证据没有把上述generic framework通过Ascend kernel owner、metadata/cache ops和sparse MLA/DSA backend串成可执行910C链路。
- **GLM-5.2 Ascend E2E Indexer — Missing：** GLM仍被sparse MLA/DSA selector与backend闭环阻塞，当前910C CI也没有GLM/Indexer case。

因此“Indexer Missing”只适用于目标平台闭环/E2E，不能解释成仓库没有Indexer framework。

## W8A8

- Current main未找到`quant_model_description.json` reader。
- **Compressed-tensors contract — upstream-owned / target unverified：** current new main已移除v0.2.1-era FL custom validator，依赖vLLM0.24/compressed_tensors contract；必须用真实artifact审计。
- **Historical packed glue — maintenance reference only：** v0.2.1有`CompressedTensorsPackedW8A8Int8`，但current main `vllm_fl/quantization/`仅剩`quant_linear.py`，不能假设old loading/packed glue仍生效。
- **Historical 0.20.2 OOT/NPU INT8 candidate — Missing：** old selector仅有CPU/CUDA/ROCm。Primary已切换0.24，必须重新审查vLLM selector、FL glue、FlagGems v5.3.4 `matmul_int8`、vendor.ascend与NPU reference；当前状态为Unknown/unverified，不能把old表直接当0.24结论。
- **Ascend 910C W8A8 Linear runtime — Missing/unverified closure：** contract/loading pieces存在，但new-main NPU kernel selection与910C E2E未闭合。
- W8A8 MoE经FL oracle到vLLM functional Triton experts，只有unit/mock，无910C E2E。
- ModelSlim是offline producer，不是当前runtime dependency；能量化不等于FL能加载。

## MoE

- Router：vLLM GateLinear + FL OOT/dispatch，FlagGems或pure torch。
- BF16 experts：FL Ascend adapter优先torch-npu grouped matmul/MoE ops，必要时Python fallback；Qwen3.6-35B-A3B确认基础链。
- EP dispatch/combine：vLLM实现；current CI无EP/multinode。
- W8A8 experts不走上述unquantized adapter，必须独立验证。

## “FlagOS原生”判定

当前验收定义改为**运行时ownership**：`PlatformFL -> WorkerFL -> ModelRunnerFL -> FlagOS Dispatch`必须真实激活；每个关键operator必须记录最终选择的FlagGems、`vllm_fl...vendor.ascend`或Reference实现及其torch_npu/CANN下游。vllm-ascend image/package存在只作environment inventory，不自动判合规或违规。

static scan在FL v0.2.1与observed new main中未发现把`vendor.ascend`实现成vllm-ascend wrapper的证据，但这不能替代进程级trace。New v0.24 A2尚未Ready；Eager Correctness后的A/B审计才回答coexistence动态参与。若发现实际参与，必须对具体调用单独裁定；不能把整个carrier先验拒绝。

部分`vendor.ascend`文件明确标注`Adapted from vllm-ascend`，但当前owner、module path和直接调用均位于`vllm_fl`。源码provenance继续遵守license/attribution；只有客户另行禁止official FL历史adapted来源时才扩大合规判断。

当前关键源码证据：[new-main FL entry points](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/pyproject.toml#L36-L40)、[`PlatformFL`](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/vllm_fl/platform.py)、[`WorkerFL -> ModelRunnerFL`](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/vllm_fl/worker/worker.py)、[Ascend per-op policy](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/vllm_fl/dispatch/config/ascend.yaml)、[`vendor.ascend` attention selector](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/vllm_fl/dispatch/backends/vendor/ascend/ascend.py)、[direct torch_npu attention implementation](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/vllm_fl/dispatch/backends/vendor/ascend/impl/attention.py)。Current main `docker/ascend/Dockerfile`是stale conflict，不作为0.24 carrier证据。
