# FlagOS on Ascend 910C Runtime Ownership Map

固定代码：FL `38e7dbc` + official vLLM `bc150f5`（v0.20.2）

## 真实调用链

```text
vllm.LLM / vllm serve
  -> vLLM EngineArgs / EngineCore / Scheduler / Executor
  -> platform entry point fl=vllm_fl:register
  -> PlatformFL(device=npu, dist=hccl, worker=WorkerFL)
  -> WorkerFL (NPU/distributed init, FlagGems enable, Ascend patches)
  -> ModelRunnerFL
  -> vLLM model loader / ModelRegistry
  -> vLLM model class
  -> vLLM layers + FL OOT layers + CachedOp dispatch
       -> FlagGems/Triton
       -> FL vendor.ascend adapter -> torch-npu -> CANN/AICore
       -> reference fallback where allowed
  -> HCCL default / FlagCX optional
  -> Ascend 910C
```

## 接管边界

- vLLM拥有API、serve、scheduler、EngineCore、Executor、model loader/registry与GLM模型主体。
- FL接管platform registration、Worker、ModelRunner、OOT layers与operator dispatch。
- `WorkerFL`/`ModelRunnerFL`是FL维护的vLLM fork，不是vllm-ascend类。
- Dense Attention glue属于FL，kernel owner是torch-npu/CANN。
- FlagGems负责ATen替换与部分activation/router/operator；不拥有当前Ascend dense attention，也没接通sparse MLA。
- FlagTree/Triton是compiler/runtime provider，不是model/attention/quant owner。
- FlagCX是optional collective/KV transfer；默认是HCCL。

## Ownership matrix

| 模块 | Owner / 文件或类 | 下游 | FlagOS原生 | torch-npu/CANN | 历史vllm-ascend | 成熟度 |
|---|---|---|---|---|---|---|
| API/LLM/serve | vLLM entrypoints | EngineCore | 否 | 间接 | 否 | Confirmed CI |
| Scheduler/Engine/Executor | vLLM `v1/engine` | WorkerWrapper | 否 | 间接 | 否 | Confirmed CI |
| Platform registration | FL `register` / `PlatformFL` | vLLM config/platform | 是 | 探测间接 | OOT设计相关 | Confirmed |
| Worker | FL `WorkerFL` | ModelRunnerFL | FL-owned fork | 是 | 无direct import | Confirmed Qwen910C |
| ModelRunner | FL `ModelRunnerFL` | vLLM model | FL-owned fork | 是 | 无direct import | Confirmed Qwen910C |
| Loader/registry | vLLM | Model class | 否 | 间接 | 否 | Confirmed |
| GLM config bridge | FL `GlmMoeDsaConfig` | transformers/vLLM | 是 | 否 | 否 | Implemented, no GLM E2E |
| GLM model class | vLLM DeepSeekV2-derived | MLA/Indexer/MoE | 否 | 间接 | 否 | 5基础存在；5.2语义Missing |
| OOT layers | FL `ops/custom_ops.py` | FL dispatch | 是 | 视op | 否 | Confirmed framework |
| CachedOp dispatch | FL `dispatch` + `ascend.yaml` | flagos/vendor/reference | 是 | 视backend | 否 | Confirmed framework |
| Dense attention glue | FL `vendor/ascend/impl/attention.py` | torch-npu ops | adapter是 | 强 | 明确adapted | Confirmed Qwen910C |
| Dense attention kernel | torch-npu/CANN | CANN/AICore | 否 | 核心 | API历史相关 | Confirmed Qwen910C |
| Dense MLA | 无usable owner；placeholder | — | 否 | — | 对照实现未迁入 | **Missing** |
| Sparse MLA/DSA/SFA | Ascend selector显式拒绝 | — | 否 | — | 历史实现存在于vllm-ascend | **Missing** |
| IndexShare logic | vLLM 0.23+；0.20.2缺 | Indexer | 否 | 间接 | 否 | **Missing baseline** |
| Indexer NPU kernel wiring | 无；upstream只CUDA/ROCm/XPU | — | 否 | — | 否 | **Missing** |
| ATen replacement | FlagGems enable + blacklist | Triton/torch fallback | 是 | fallback依赖 | 否 | Worker confirmed；覆盖有限 |
| RMSNorm/RoPE | FL Ascend adapter | torch-npu | adapter是 | 强 | RoPE有历史来源 | Confirmed链路 |
| Router | vLLM + FL GateLinear/router patches | FlagGems/pure torch | 混合 | 间接 | 否 | BF16 MoE confirmed |
| BF16 experts | FL `impl/fused_moe.py` | NPU grouped matmul/MoE | adapter是 | 强 | 未标direct | Qwen MoE confirmed；分支Unknown |
| EP dispatch/combine | vLLM managers | HCCL/FlagCX | 否 | backend相关 | 否 | Implemented, unverified EP |
| W8A8 metadata | vLLM compressed-tensors | INT8 schemes | 否 | 间接 | 否 | Contract confirmed |
| AscendV1 description | FL无人读取；reader仅在vllm-ascend | — | 否 | — | 被禁package独有 | **Missing** |
| W8A8 Linear | FL candidate list均拒绝NPU | — | 否 | — | 否 | **Missing** |
| W8A8 MoE | FL oracle + vLLM Triton experts | Triton provider | 混合 | 预期 | 否 | Implemented, unverified910C |
| TP communication | torch distributed/HCCL | HCCL | 否 | 强 | 否 | TP2 Confirmed |
| FlagCX collective | FL communicator + FlagCX | adaptor/HCCL | FlagOS组件 | adaptor依赖 | 否 | Optional, unverified910C |
| P/D KV transfer | FL `FlagCXConnector` | libflagcx P2P | FlagOS组件 | adaptor依赖 | 否 | Implemented, unverified |
| Triton compiler | triton-ascend(CI)或FlagTree(intended) | CANN codegen | profile相关 | 是 | 生态相关 | CI profile confirmed；FlagTreeUnknown |
| A3/910C FL extension | 无；`VLLM_VENDOR`只支持cuda | 下游二进制栈 | 否 | 依赖下游 | 否 | Not designed |

## Attention

- Dense：`PlatformFL -> vendor.ascend -> torch_npu`；Qwen3.6 TP2 910C E2E Confirmed。
- MLA：Ascend返回构造即报错的placeholder；Missing。
- DSA/SFA：`use_mla && use_sparse`直接 `NotImplementedError`；Missing。
- FlagGems内有DSA相关kernel不等于plugin backend可达；selector/metadata/cache/backend必须闭合。

## W8A8

- 固定树没有 `quant_model_description.json` reader。
- FL只接compressed-tensors；Linear candidates按CUDA条件筛选，NPU无候选。
- W8A8 MoE经FL oracle到vLLM functional Triton experts，只有unit/mock，无910C E2E。
- ModelSlim是offline producer，不是当前runtime dependency；能量化不等于FL能加载。

## MoE

- Router：vLLM GateLinear + FL OOT/dispatch，FlagGems或pure torch。
- BF16 experts：FL Ascend adapter优先torch-npu grouped matmul/MoE ops，必要时Python fallback；Qwen3.6-35B-A3B确认基础链。
- EP dispatch/combine：vLLM实现；current CI无EP/multinode。
- W8A8 experts不走上述unquantized adapter，必须独立验证。

## “FlagOS原生”判定

建议验收定义：不安装、不发现、不import、不激活、不链接vllm-ascend package/runtime；允许official FL仓库内已维护adapter和torch-npu/CANN下游。按此定义，`R0-clean`可通过实验成为FlagOS-native。

如果客户也禁止任何historical adapted/copied源码，则current official FL main本身不合规，必须在执行前裁定。
