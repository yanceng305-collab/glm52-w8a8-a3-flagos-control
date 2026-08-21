# A3-CP-A2-v024 — Official v0.24 Carrier + FlagOS main FL-only Environment Smoke

状态：**Superseded by final Ready contract**
执行授权：无
第二台A3：不需要；第一台A3至少需要一个完整、明确空闲的valid logical-device pair

## Objective

> 本draft已由[`STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)取代，仅保留审计历史，不得执行。

使用本机已有`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`作为environment carrier，在一次性实验container中构造FlagOS new-main FL-only环境，并完成最小platform/worker/model-runner/dispatch与synthetic NPU operator smoke。

本任务不加载Qwen/GLM，不验证mandatory GLM capability，不做benchmark/profile，也不做完整coexistence provenance。

## Frozen research inputs

- FL new-main research freeze：`a9435a34dcd7d0a38e3a853535947371a6c62205`，tree`e5e073edf4b65c053e954d78d20365aab0e1f46b`；repo migration/task Ready前必须重读。
- vLLM 0.24.0。
- vLLM-Ascend `v0.24.0rc1@412cda26814ff70c326f6eb6510f1b610f67bbc0`。
- Carrier source tuple：CANN9.0.1 / Ubuntu22.04 / Python3.12 / torch2.10.0 / torch-npu2.10.0.post2 / vLLM0.24 empty / triton-ascend3.2.1 / transformers5.13.0 / `compressed_tensors>=0.11.0` / `SOC_VERSION=ascend910_9391`。
- FlagGems `v5.3.4@f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`。
- Intended Ascend README provider：FlagTree `0.6.1rc1+ascend3.5@9a90fddf166d33217777b662821072c41015b294`。
- `TRITON_ALL_BLOCKS_PARALLEL=1`；eager execution。
- FlagCX/FlagScale后置。

## Ready blockers

1. 用户尚未批准正式代码repo的0.24 branch migration；当前正式repo只有v0.2.1 baseline。
2. mutation时official main exact SHA/tree尚未最终冻结。
3. FlagTree与carrier `triton-ascend`的replacement/overlay transaction尚未闭合，不能保证single coherent `triton` provider。
4. resource index FlagTree wheel的exact provenance/RECORD/hash与安装影响Unknown。
5. 本机v0.24 carrier RepoDigest/image ID/actual inventory未知；后续只允许检查已有image，不得pull。
6. A3至少双logical-device；合法pair mapping及OC2占用需server preflight。没有完整free pair即STOP。

全部blocker关闭且control重新批准后，才能生成新的DeepSeek prompt。

## Device rule

- 不允许单logical-device A2。
- 至少两个logical devices。
- 必须以Host只读拓扑证据证明它们构成valid A3 working combination。
- 优先同一physical card pair，但official文档未定义ID映射，不得按`(0,1)`等模式猜测。
- 重新检查OC2/其他进程；任一device被占用、pair不完整或映射不确定即STOP。

## Compiler/provider resolution gate

Carrier起点为`triton-ascend==3.2.1`；FL new-main README intended profile为FlagTree `0.6.1rc1+ascend3.5`。两者都拥有完整`triton`namespace，不是可直接并排使用的两个provider。

未来A2 contract必须先定义并验收：

1. pre-transaction distributions、RECORD、`triton`module/native extension/backend origin；
2. FlagTree wheel exact identity与离线可用性；
3. 安装计划不得联网解析或改动torch/torch-npu/vLLM/CANN等非目标package；
4. post-transaction只能有一个coherent `triton`file tree/provider；
5. 若同时残留互相冲突的dist-info/RECORD或混合file ownership，STOP，不手工删除修补；
6. provider性能与完整dynamic provenance可以后置，但A2必须知道synthetic operator实际使用哪个provider。

## FL installation rule

正式source repo始终readonly。若需安装new-main FL：

1. 完整复制到container writable staging并保留`.git`；
2. 安装前验证task冻结的exact HEAD/tree/clean；
3. 只允许等价：`python -m pip install --no-build-isolation --no-deps -e <writable-staging>`；
4. `_version.py`、egg-info与build artifacts只留container副本；
5. 缺少明确build requirement时STOP，不得自行联网补包。

## Intended smoke scope

在compiler/profile与repo baseline闭合后，新A2仍只验证：

- exact local carrier启动；
- container内移除vllm-ascend plugin并在fresh Python process做minimal negative check；
- torch/torch-npu与valid A3 pair可用；
- `VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`；
- `PlatformFL`、`WorkerFL`、`ModelRunnerFL`；
- FlagOS Dispatch和effective Ascend policy；
- 一个最低风险、小shape、NPU-resident synthetic operator；
- tensor保持NPU，无silent CPU fallback；
- Evidence位于task-start WORKDIR的新目录。

## Stop / prohibited

- 不操作服务器（本轮）；
- 不pull/build/commit image；
- 不修改Host Driver/Firmware/CANN/network；
- 不kill任务或访问第二台；
- 不修改正式代码repo；
- 不加载模型；
- 不进入MLA/DSA/Indexer/W8A8；
- 不benchmark/profile；
- 不生成DeepSeek prompt；
- 不在Unknown compiler transaction下安装FlagTree。

## Draft PASS definition

最终A2 PASS仍只证明v0.24 carrier + FL new-main最小环境闭环，不证明Qwen、GLM-5.2-W8A8或完整runtime provenance。实际PASS/STOP细节须在上述Ready blockers关闭后由control再次冻结。
