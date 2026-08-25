# CANARY-V024-QWEN36-27B-TP2

状态：**SUPERSEDED / CANCELLED UNEXECUTED by User Decision；historical contract only；never dispatched；must not be executed**

本合同未被 User下发、没有 run/Evidence/Code change/Acceptance。不得通过填写旧占位符或复用历史 dispatch恢复执行；未来 GLM恢复必须经过新的 User Decision、vLLM 0.20.2 GLM contract review和新 Task。

执行代理：Codex2

目标硬件：第一台Ascend A3/910C服务器，单机TP2

默认runtime：`flagos-glm52-a3-runtime:v024-cann900-py311`

## User-confirmed facts（Ready前必须填写）

- Exact local model path：`<USER MUST CONFIRM>`
- Authorized safe 910C logical devices：`<USER MUST CONFIRM OR EXPLICITLY AUTHORIZE TASK-SAFE SELECTION>`
- User dispatch：`<NOT DISPATCHED>`

未填写以上字段时，合同存在不等于Ready；Codex2不得执行。

## Frozen model decision

| Field | Value |
|---|---|
| Model | `Qwen/Qwen3.6-27B` |
| Official ModelScope | <https://www.modelscope.cn/models/Qwen/Qwen3.6-27B> |
| Complete repository revision | `cea40373b9214dd387123e68841890af30dcd469` |
| Payload revision | `c53c4820996523bb6413f1002e24c5dfb0bad548` |
| Artifact | 完整official standard BF16 repository；非量化、非FP8/W8A8/GGUF，不允许partial shards |
| Expected identity | `Qwen3_5ForConditionalGeneration` / `model_type=qwen3_5`；15个safetensors shards，weight payload `55,562,855,904` bytes |
| Preferred path | `/data/tiankuan/zyg/artifacts/models/Qwen/Qwen3.6-27B/cea40373b9214dd387123e68841890af30dcd469/` |
| Existing-copy exception | 已有`/data/models/Qwen/Qwen3.6-27B`可在User冻结该path且read-only核对匹配上述revision/config/index/shard identity后复用 |

Codex1与Codex2不得下载、修复、转换、量化或替换模型。缺失/不完整/identity不匹配时保持Not Ready或执行后STOP，等待User处理；不得自行改用Qwen3.6-35B-A3B、Qwen3-4B或其他模型。

## 选择依据

- Qwen3.6-27B dense TP2保留official FL历史910C成功case作为同model/hardware oracle，使v0.24 runtime迁移成为主要变量；该历史case只作reference，不替代本task验证。
- 相比Qwen3.6-35B-A3B，它不引入256 experts、router/shared expert、FusedMoE/EP/provider等额外变量，权重更少且更易归因。
- Qwen3-4B虽更小且在frozen v0.24 tests中有配置，但对应Ascend matrix是910B serving而非同等级910C oracle，并不能规避当前共同full-attention construction风险；它不是自动fallback。

## 目标

在accepted v0.24 snapshot和exact clean FL baseline上，对选定本地完整BF16 checkpoint完成一次TP2/HCCL、text-only、eager offline deterministic generation，并用同一formal run证明：

```text
vLLM 0.24 model registry/config/loader
  → PlatformFL
  → WorkerFL（两个TP rank）
  → ModelRunnerFL
  → Qwen3.6-27B构造与真实权重加载
  → FlagOS Dispatch / actual Ascend providers
  → torch_npu / FlagTree / CANN
  → Ascend 910C NPU
```

Task只需PASS该闭环，或在第一个可归因blocker处STOP并保留证据。

## Frozen runtime / Code

- Snapshot tag/image ID：`flagos-glm52-a3-runtime:v024-cann900-py311` / `sha256:e1a89dca8f2580298842d5de3745cc674feea348ab272fd1ab94779542afbd20`。
- Runtime tuple与launch边界以[`../A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md`](../A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md)为准。
- Code repo：`yanceng305-collab/vllm-plugin-FL-a3-flagos`。
- Branch/commit/tree：`project/glm52-w8a8-v024` / `a9435a34dcd7d0a38e3a853535947371a6c62205` / `e5e073edf4b65c053e954d78d20365aab0e1f46b`，必须clean；Canary Code PR=`N/A`。
- Evidence：`/data/tiankuan/zyg/evidence/CANARY-V024-QWEN36-27B-TP2/<run-id>/`。
- Work：`/data/tiankuan/zyg/work/CANARY-V024-QWEN36-27B-TP2/<run-id>/`。

## Ready gate

全部满足后才可由User下发：

1. A2 Acceptance仍有效；snapshot image ID、runtime tuple、required mounts和import origins重新核对一致。
2. User确认一个exact absolute model path和上述ModelScope revision；read-only检查证明config/index/tokenizer与15个shard完整、非空并形成file/size/SHA256 manifest，且BF16/no quantization identity一致。
3. User确认两个安全的910C logical devices或明确授权task内安全选择；内存、HCCL topology和现有任务边界允许TP2，不得干扰其他工作。
4. `/data`下formal FL checkout为exact branch/commit/tree/clean，snapshot启动后实际import指向该checkout。
5. Effective Canary配置已冻结，且无需remote code、其他model/provider、依赖/runtime或FL修改。
6. User明确下发本Task ID。

## Minimal run shape

- 本地path、local-files-only；不访问remote model code/hub。
- Official BF16 real weights；禁止dummy load、quantized/derived checkpoint或CPU offload。
- Text-only；vision/multimodal limits为零，vision tower不参与。
- TP=2、HCCL、恰好两个获授权910C logical devices；不启用FlagCX或多机。
- Eager offline为mandatory；graph、MTP/speculative、multistream、prefix caching、async scheduling、chunked prefill和performance feature均关闭。
- Bounded configuration：`max_model_len=4096`、memory-utilization ceiling `0.8`、seed `0`、temperature `0`、最多16个output tokens。
- Frozen oracle：prompt `The capital of France is`；必须完成真实prefill和multi-token decode，生成非空finite output且包含`Paris`。
- Minimal serving默认`N/A`，不属于PASS。只有offline PASS后、且Control明确认为需要service-entry proof时才允许独立的bounded serving sub-gate；不得扩张为benchmark/production claim。

普通安全检查、执行顺序和排障方式由Codex2在以上边界内决定。

## Known first-blocker risks

以下是静态风险，不是预判的execution结果：

1. `AscendAttentionBackend.get_name()`返回`ASCEND_FL`，而vLLM0.24 `AttentionBackendEnum`没有同名member；first full-attention layer可能在weight load前以`KeyError: ASCEND_FL`失败。
2. vLLM0.24 Qwen GDN callsite已移动，当前Ascend patch/import binding可能未覆盖live GDN prefill/decode provider。
3. TP2/HCCL model-level collective、55.56GB BF16 load headroom、text-only wrapper和no-CPU-fallback尚未经本v0.24 tuple验证。

若风险真实触发，保存first blocker并STOP。Canary task不得patch FL；需要Code修改时提出`Decision requested`，由后续独立Code task/branch/PR处理。

## PASS

同一formal run必须全部证明：

- snapshot/runtime、Code和model repo/revision/path/manifest identity exact；
- vLLM内置`Qwen3_5ForConditionalGeneration`识别、config解析和text-only model construction成功；
- 两个rank的`PlatformFL`、`WorkerFL`、`ModelRunnerFL`实际class/module origin明确，TP2/HCCL rank-device mapping正确；
- complete real BF16 weights全部加载，无unexpected core missing weights；核心parameters位于NPU；
- 至少一个GDN layer和一个full-attention layer完成实际prefill/decode，记录live Dispatch/backend/provider/function、shape/dtype/device/finite assertions；
- core parameters、GDN/full-attention tensors、cache/state和logits compute均在NPU，无silent CPU fallback；tokenization/scheduler metadata和最终返回token IDs的正常CPU边界须单独区分；
- frozen prompt完成真实multi-token eager generation，exit=0，输出非空、finite、无乱码并包含`Paris`；
- FL和runtime未修改，Code PR=`N/A`；Evidence manifest/checksum与Code/Control/Evidence三指针完整。

## STOP

在第一个有用failure处STOP，如果：

- model path/revision/config/index/tokenizer/shards不完整或不匹配；
- snapshot/runtime/Code/provider identity漂移，或两device/HCCL资源边界不安全；
- architecture construction、real weight load、TP/HCCL、GDN/full attention、Dispatch ownership、NPU execution或correctness失败；
- 出现CPU fallback、non-finite output、错误backend或输出不满足oracle；
- 继续需要download/remote code、模型替换、更多device、dependency/runtime mutation、FL修改或无关retry matrix。

STOP须保存first exception、inner traceback、last successful gate、live class/provider/device、effective config和numeric exit。不得在本task内修Code或继续到serving。

## Evidence

- Task/run、snapshot/container、runtime/provider、Code三指针和pre/post resource inventory；
- official ModelScope repo/revision/local path、config/index/tokenizer、15-shard file/size/SHA256 manifest、BF16/no-quantization证明；
- exact effective config/selectors、TP2/HCCL world/rank/device mapping；
- 每rank的Platform/Worker/Runner/model wrapper/inner LM class与module origin；
- construction和complete real-weight load logs、missing/unexpected key summary、parameter dtype/device；
- 一个GDN与一个full-attention prefill/decode的minimal live trace，以及Dispatch/backend/provider/function和no-CPU-fallback assertions；
- prompt/sampling、raw token IDs/text、`Paris` assertion、NPU synchronization、numeric exit；
- pre/post occupancy和peak memory（只作安全证据，不形成performance baseline）；
- STOP时first blocker最小上下文；
- raw logs、manifest、SHA256及成功checksum verification。

不得把Evidence扩张为deferred exhaustive provenance audit。

## Codex1 Acceptance boundary

Execution PASS不自动等于Acceptance。Codex1独立审查三指针和Evidence后，`ACCEPTED`只表示：accepted v0.24 snapshot + frozen FL能识别、构造、完整加载并在910C TP2 eager offline执行exact Qwen3.6-27B BF16 checkpoint，且live Platform/Worker/Runner/Dispatch链在NPU闭合并通过固定smoke oracle。

它不接受：GLM-5.2-W8A8、W8A8/quantization、MLA/DSA/Indexer/KV cache capability、广义model correctness、production serving、graph/MTP/FlagCX/multistream、多机、benchmark、profiling、performance、稳定性或exhaustive provenance。
