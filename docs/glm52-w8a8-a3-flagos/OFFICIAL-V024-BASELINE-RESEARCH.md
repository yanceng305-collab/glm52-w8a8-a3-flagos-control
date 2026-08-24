# Official v0.24 Baseline Refresh Research

研究冻结时间：2026-08-21；artifact decision更新：2026-08-24
执行边界：仅official源码/文档/GitHub元数据；未访问服务器、未pull image、未修改正式代码repo

## 结论

本项目primary development line改为official `flagos-ai/vllm-plugin-FL` new `main` + vLLM 0.24.0。本轮观测冻结：

- FL `main`：`a9435a34dcd7d0a38e3a853535947371a6c62205`
- FL `main` tree：`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FL `v0.2.1`：`92a6f7670465922c60e88f06787b8f0923e761f3`
- FL `v0.2.1` tree：`e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`

`main`是移动分支；任何未来正式repository mutation前必须重新读取并冻结当时exact SHA/tree。本轮SHA是research freeze，不是永久别名。

## Official branch migration

| 项目 | 当前事实 | 状态 / 来源 |
|---|---|---|
| Default branch | `main` | Confirmed，official GitHub repository metadata |
| `main`语义 | vLLM 0.24.0 current/development line | Confirmed：客户/FlagOS开发者通知（user-confirmed）+ current pyproject/README/source |
| `v0.2.1`语义 | vLLM 0.20.2 maintenance/reference line | Confirmed：开发者通知 + branch HEAD + [`pyproject.toml`](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/pyproject.toml#L70) |
| `v0.3.0-dev` | 无独立active Git ref；old-name lookup解析到`main` | Confirmed：branch/ref API；PR自动迁移由开发者通知确认 |
| `main` / `v0.2.1`关系 | Diverged；merge base `dbfe3be58ae9fa1eb2b0e9bc0103fb0d52c51184` | Confirmed：[compare](https://github.com/flagos-ai/vllm-plugin-FL/compare/92a6f7670465922c60e88f06787b8f0923e761f3...a9435a34dcd7d0a38e3a853535947371a6c62205) |

因此`92a6f767...`没有失效，而是被正式重分类为`v0.2.1`维护线；new main不能作为它的fast-forward升级，也不能通过force-push覆盖当前正式repo `main`。

## New main vLLM/component contract

### Confirmed

- current [`pyproject.toml`](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/pyproject.toml#L45-L56)固定`vllm[audio]==0.24.0`。
- current [`README`](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/README.md#L42-L51)要求non-NVIDIA使用official vLLM v0.24.0 empty build。
- FL platform/general entry points仍为`fl = vllm_fl:register` / `register_model`。
- current main仍静态进入`PlatformFL -> WorkerFL -> ModelRunnerFL -> FlagOS Dispatch`。
- README选择FlagGems `v5.3.4`；tag精确指向`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`，tree `87e4e1e98c80dfd31d923bd726795f385aa28ffd`。
- Ascend additional setup选择FlagTree `0.6.1rc1+ascend3.5`；Git tag指向`9a90fddf166d33217777b662821072c41015b294`，tree `0b9b43211eb176a3a750e07fdc579f6322a94a66`。
- README要求`TRITON_ALL_BLOCKS_PARALLEL=1`与eager execution。
- FlagCX仍optional/后置，不是first eager前置。

主要来源：[FL README](https://github.com/flagos-ai/vllm-plugin-FL/blob/a9435a34dcd7d0a38e3a853535947371a6c62205/README.md)、[FlagGems tag](https://github.com/flagos-ai/FlagGems/commit/f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9)、[FlagTree tag](https://github.com/flagos-ai/FlagTree/commit/9a90fddf166d33217777b662821072c41015b294)。

## Official GLM-5.2 v0.24 evidence

Commit [`bb439d028479475a965712e08ce0b955fe02aafb`](https://github.com/flagos-ai/vllm-plugin-FL/commit/bb439d028479475a965712e08ce0b955fe02aafb)是observed `main`的祖先；GitHub compare显示`main`领先17个commit、behind 0、merge base即`bb439d...`。

该commit明确记录：

| 项目 | 结果 | 状态/边界 |
|---|---|---|
| Model | GLM-5.2-Slim | Confirmed |
| Type | Dense MoE (DSA/MLA) | 按commit原文记录 |
| Layout | TP16 / 2-node | Confirmed |
| Overall | PARTIAL | Confirmed |
| 2-node init | PASS | NVIDIA A800 / NCCL only |
| Weight loading | PASS | NVIDIA A800 only；非目标W8A8/910C证据 |
| First gap | `concat_and_cache_mla`缺失 | Confirmed at KV-cache population |
| Related gap | `concat_and_cache_mla_rope_fused`缺失 | Confirmed in commit record |

这证明0.24 line已包含足够GLM model contract完成构造、双机初始化和weight loading，不再是从0.20.2 blank-slate探索；但它不证明first token、W8A8、Ascend/HCCL或910C correctness。

### Current main static GLM assessment

- vLLM 0.24注册`GlmMoeDsaForCausalLM`并包含MLA、DSA/Indexer、MoE与weight-loading结构。
- FL current main提供`GlmMoeDsaConfig` bridge、platform/dispatch与cache/indexer schemas。
- `concat_and_cache_mla`、fused RoPE variant、Indexer cache/gather等schema存在。
- current FL source未找到上述GLM路径的`vendor.ascend`闭环；upstream native SparseAttnIndexer仅声明CUDA/ROCm/XPU。
- FlagGems v5.3.4存在generic Triton `concat_and_cache_mla`，但current FL -> FlagGems -> Ascend的可达性和910C correctness仍Unknown。
- current `vendor.ascend`仍显式拒绝sparse MLA，MLA class仍为placeholder。
- v0.2.1-era FL `compressed_tensors.py`与`w8a8/packed.py`不在observed new-main tree；current `vllm_fl/quantization/`仅保留`quant_linear.py`。W8A8 contract/loading ownership必须按vLLM0.24 upstream与真实artifact重新冻结，不能沿用old glue结论。
- 未找到GLM-5.2-W8A8 × FlagOS main × Ascend 910C E2E证据。

结论：primary起点前移到0.24，但MLA/DSA/SFA/Indexer/W8A8与MLA cache ops仍必须进入新的mandatory capability gap confirmation。

## Documented A3 release tag and provisional A2 carrier

Official文档定义`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`，但当前registry未建立可用artifact。对应[tag-push image workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/30890938662) cancelled，不能把文档tag当作可拉取或已冻结的release image。

A2当前唯一provisional carrier为`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`；其[A3 official workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/30602986508)成功完成。该artifact身份只能写为official `releases/v0.24.0rc` A3 nightly，不得称为rc1 release image或其alias。

Official vLLM-Ascend source tag：

- tag `v0.24.0rc1`
- commit `412cda26814ff70c326f6eb6510f1b610f67bbc0`
- tree `92a290701c2df5ec62bc92c19923aaadab27c7c9`

### Documented rc1 source-defined tuple

| Component | Value | Status |
|---|---|---|
| Base/OS | `quay.io/ascend/cann:9.0.1-a3-ubuntu22.04-py3.12` / Ubuntu 22.04 | Confirmed source contract |
| Python | 3.12 | Confirmed base-tag contract |
| CANN | 9.0.1 A3 | Confirmed base-tag contract |
| vLLM | v0.24.0, observed tag`ee0da84ab9e04ac7610e28580af62c365e898389`, `VLLM_TARGET_DEVICE=empty` | Version/build contract Confirmed；actual image embedded SHA待验 |
| vLLM-Ascend | v0.24.0rc1 source/custom kernels | Confirmed tag/build contract |
| SOC | `ascend910_9391` | Confirmed Dockerfile |
| torch | 2.10.0 | Confirmed requirement |
| torch-npu | 2.10.0.post2 | Confirmed requirement |
| torchvision / torchaudio | 0.25.0 / 2.10.0 | Confirmed requirement |
| triton-ascend | 3.2.1 | Confirmed starting provider |
| transformers | 5.13.0 | Confirmed requirement |
| compressed_tensors | `>=0.11.0` | Confirmed constraint；exact resolved version Unknown |
| NNAL/ATB | CANN 9.0.1 family | Version contract known；actual carrier inventory Unknown |

Sources：[Dockerfile.a3](https://github.com/vllm-project/vllm-ascend/blob/412cda26814ff70c326f6eb6510f1b610f67bbc0/Dockerfile.a3)、[requirements.txt](https://github.com/vllm-project/vllm-ascend/blob/412cda26814ff70c326f6eb6510f1b610f67bbc0/requirements.txt)。该表只描述documented rc1 source contract，不定义provisional nightly的实际filesystem/package tuple。

### Artifact evidence boundary

Official versioned docs列出release tag，但tag-push image workflow cancelled，registry当前无该artifact。服务器本地已有上述exact nightly digest；A2只接受该digest并冻结image ID/architecture。其OS/Python/CANN、torch/torch-npu、vLLM、compiler/provider及其他package版本必须由container preflight实测，不得从rc1 source contract外推；digest缺失或不匹配即STOP。

## A3 device requirement

- **Confirmed：** vLLM-Ascend v0.24 Quick Start明确“A3 requires at least 2 NPUs to work together”，示例同时挂载两个`/dev/davinci`节点：[quick start](https://github.com/vllm-project/vllm-ascend/blob/412cda26814ff70c326f6eb6510f1b610f67bbc0/docs/source/quick_start.md#L89-L105)。
- **Confirmed project topology：** 8 physical cards -> 16 logical devices -> 16×64GB -> 1024GB；不重新打开该事实。
- **Inferred：** 新A2至少暴露两个logical devices，并应选择已知有效组合。
- **Unknown：** official文档没有定义logical ID到physical card/die的通用映射，也没有明确最小pair必须是同一physical card。`davinci0/1`只是示例。
- **A2 current decision：** tiny smoke固定共享NPU 12+13；不再要求完全空闲pair。只允许极小torch_npu tensor与FlagOS Dispatch operator，开始/结束只读记录占用；状态恶化或可能干扰现有任务立即STOP。

## FlagTree / triton-ascend relationship

### Confirmed

- FlagTree distribution名为`flagtree`，但打包完整`triton`、`triton.*`、`triton/_C`和`triton.backends.ascend`。
- triton-ascend v3.2.1 tag指向`2badfc89e70a9b7a5e88463a116c2feddce4b101`；distribution同样打包完整`triton` namespace和`triton/_C`。
- 两者是同层、互斥的compiler/provider选择；运行时只能形成一个coherent `triton` file tree。
- FlagTree不是`triton-ascend`或vllm-ascend backend代理。
- `TRITON_ALL_BLOCKS_PARALLEL=1`被两种provider共同读取，不能用于识别active provider。

### Replacement/overlay assessment

若provisional carrier实际含与FlagTree共享namespace的provider，安装FlagTree应理解为compiler replacement/overlay transaction，而非clean coexistence。A2必须先inventory实际distribution/version/RECORD；不得把rc1 recipe中的`triton-ascend==3.2.1`当作该nightly的既定starting state。

### Unknown / A2 runtime gate

- resource.flagos.net wheel与Git tag的exact build provenance/RECORD/hash；
- 安装事务是否移除或保留`triton-ascend`metadata；
- 安装后每个`triton/*`文件owner、`triton.__file__`、version、backend entry point、active driver/provider、native extension origin；
- FlagGems v5.3.4 + FlagTree rc1 + FL new main在A3上的official E2E。

### Upstream profile conflict

FlagGems v5.3.4 generic `ascend-cann900` profile仍列Python3.11和FlagTree`0.6.0+ascend3.5`，而更晚、面向FL integration的README选择Python3.12 carrier与FlagTree`0.6.1rc1+ascend3.5`。本项目优先研究较新的FL README intended profile，但在A3实验前保持**Conflict / runtime Unknown**，不得把两者合并成“已确认兼容”。

新的A2不预先宣称安全共存；它被授权在disposable container内执行可审计replacement transaction。无法得到单一coherent provider时A2 STOP。

## Current minimal runtime tuple candidate

`R0-v024-FL-main`已冻结为**Ready experiment profile**；下列Unknown转化为task内PASS/STOP gate：

| Layer | Candidate | Status |
|---|---|---|
| Carrier | exact `quay.io/ascend/vllm-ascend@sha256:1c36469f...` | Official release-branch A3 nightly；provisional only |
| Base runtime | 不预设 | Container preflight冻结actual OS/Python/CANN/torch/torch-npu |
| vLLM | FL要求0.24 contract；carrier actual identity待验 | Container preflight gate |
| FL | `project/glm52-w8a8-v024` | Frozen`a9435a34...`/tree`e5e073ed...` |
| FlagGems | v5.3.4 / `f7c55cb2...` | Confirmed README pin；preferred |
| Compiler | exactly one coherent Ascend provider | Required |
| Intended compiler | FlagTree`0.6.1rc1+ascend3.5` after controlled replacement | A2执行并验证single provider；失败STOP |
| Carrier compiler before replacement | 不预设 | Container inventory后按实际distribution做replacement |
| Env | `VLLM_PLUGINS=fl`, `VLLM_FL_PLATFORM=ascend`, `TRITON_ALL_BLOCKS_PARALLEL=1`, `VLLM_VENDOR`unset | Confirmed FL contract |
| Execution | eager | Confirmed README requirement |
| Communication | A2禁止HCCL/TP | Tiny smoke不执行collective |
| FlagCX/FlagScale | absent/not active | Deferred |
| Device scope | shared logical NPU 12+13 | 只允许tiny tensor/Dispatch；状态恶化STOP |

“FlagTree + 其他同namespace provider并存”仍不是合法终态；A2按actual inventory用package manager处理冲突distribution并形成single coherent FlagTree provider。Exact digest、source bundle与共享NPU状态由task preflight处理。

## Upstream Conflict: current Ascend Dockerfile

Observed current main的`docker/ascend/Dockerfile`仍声明：

- vLLM 0.19.0
- CANN 8.5.1
- Python 3.11
- triton-ascend 3.2.0
- FlagGems v5.0.0
- FlagTree 0.4.0+ascend3.2

该文件最后相关更新为2026-04-09的[`ff715df`](https://github.com/flagos-ai/vllm-plugin-FL/commit/ff715df45db46218526ecd0235d81845a3c69792)，早于2026-07-16的0.24 upgrade和2026-08-20 README refresh。

处理规则：

1. 标记`Upstream Conflict / stale Ascend Dockerfile candidate`；
2. 不用它推翻开发者branch migration通知、current pyproject或README；
3. 不把其0.19/CANN8.5 tuple作为new main primary route；
4. 等upstream更新或开发者说明；
5. 新A2以v0.24 carrier source contract、frozen main SHA与README profile为研究候选，但compiler overlay仍需先闭合。

## FL installation rule retained

未来new A2如需安装FL：

- 正式source repo只读；
- 完整复制到container writable staging并保留`.git`；
- 安装前验证exact HEAD/tree/clean；
- 只允许等价`python -m pip install --no-build-isolation --no-deps -e <writable-staging>`；
- `_version.py`/egg-info/build artifacts只留container副本；
- 缺少build requirement时STOP并记录，不得联网补包。

理由：current main build-system requires包含torch等包，默认build isolation可能创建临时环境并联网解析；`setuptools_scm.write_to`还需要可写source tree。

## Current high-impact Unknowns

1. 本机v0.24 carrier RepoDigest/image ID/architecture/actual inventory；缺失时A2允许pull唯一exact tag；
2. valid A3 logical-device pair的Host映射和OC2占用；
3. FlagTree replacement transaction与single-provider closure；
4. exact resolved `compressed_tensors`、NNAL/ATB与other carrier packages；
5. new main + FlagGems v5.3.4 + FlagTree在910C的synthetic smoke；
6. target W8A8 checkpoint manifest/format；
7. current main Ascend MLA/DSA/Indexer/W8A8/concat cache operator closure。
