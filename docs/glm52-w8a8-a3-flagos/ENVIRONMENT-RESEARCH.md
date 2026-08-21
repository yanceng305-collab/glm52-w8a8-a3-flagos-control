# FlagOS A3/910C 环境调查与证据链

调查日期：2026-08-21
当前复核 FL：`92a6f7670465922c60e88f06787b8f0923e761f3`（2026-08-21 official `main`）；早期CI调查固定SHA `38e7dbc20197e2db742c4e4c9687d36ea4df9900`仅保留其时间语境。

## 当前Container边界决策

本项目已明确采用容器化部署。Host CANN 8.5.0/9.0.1不参与R0 tuple选择，除非未来显式bind-mount Host Toolkit；原则上R0只依赖Host A3/910C、Driver/Firmware、container runtime、device/driver挂载、HCCN/HCCL网络、disk/model路径和NPU占用。

`c70aa4b`曾把neutral CANN base和vllm-ascend package缺席设为唯一合法路线；该架构前提已被本次修正Superseded。Host/Container边界仍有效，但[`R0-CONTAINER-TUPLE-RESOLUTION.md`](R0-CONTAINER-TUPLE-RESOLUTION.md)中的R0-P1/R0-F1只保留兼容性reference evidence，不再是正式执行路线。

## 总判定

- **Confirmed：** 官方 current Ascend Dockerfile直接以`quay.io/ascend/vllm-ascend:v0.20.2rc1-a3`作环境carrier；其中包含vllm-ascend distribution/custom artifacts，而非中性CANN-only底座。
- **Confirmed：** FL Docker/CI setup 不卸载、覆盖或移除 vllm-ascend；package/entry point仍 installed/discoverable。
- **Confirmed：** Dockerfile设置`VLLM_PLUGINS=fl`与`VLLM_FL_PLATFORM=ascend`；公开CI log显示`Platform plugin fl`激活；静态源码把worker设为`WorkerFL`并构造`ModelRunnerFL`。
- **Confirmed：** official `ascend.yaml`是per-op策略：attention/rms_norm为`vendor -> flagos -> reference`，silu为`flagos -> vendor -> reference`；不是一个统一vendor backend总接管。
- **Confirmed（静态）：** `vendor.ascend`返回`vllm_fl.dispatch.backends.vendor.ascend.impl.*`类/函数并直接调用torch_npu/PyTorch；FL树未发现direct `import vllm_ascend`。部分文件明确保留`Adapted from vllm-ascend`来源，这属于源码provenance，不等于动态runtime依赖。
- **Unknown / deferred：** official coexistence进程中是否有任何`vllm_ascend`模块、native artifact或side effect实际参与执行；该问题由Post-Eager Runtime Provenance Audit按需验证，不阻塞A2、canary、first eager或Baseline Benchmark。
- **Superseded：** “客户合规第一候选必须是neutral CANN base且从未安装vllm-ascend”的推断不再有效。

## 官方实际 CI 证据链

```text
Host A3/910C runner
  executed arch: aarch64 (Confirmed)
  kernel: 5.10.0-216.0.0.115.oe2203sp4.aarch64
  driver/firmware: host mounted, exact Unknown
    ↓
quay.io/ascend/cann:9.0.0-a3-ubuntu22.04-py3.11
  Ubuntu 22.04 + Python 3.11.15 + CANN 9.0.0 + ATB paths
    ↓
official vLLM 0.20.2 source, VLLM_TARGET_DEVICE=empty
    ↓
vllm-ascend 0.20.2rc1 editable + A3 custom kernels
  torch/torch-npu 2.10.0 + triton-ascend 3.2.1
    ↓
FL layer: FlagGems 3e6528cf + VLLM_PLUGINS=fl
    ↓
CI setup: pip install --no-deps -e FL; no uninstall/replace
    ↓
 910C: carrier package discoverable, FL activated, Qwen3.6 TP2 success
```

关键证据：

- [FL Ascend Dockerfile](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/docker/ascend/Dockerfile)
- [FL Ascend setup](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/.github/scripts/ascend/setup.sh)
- [FL Ascend CI config](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/.github/configs/ascend.yml)
- [FL 910C matrix](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/tests/platforms/ascend.yaml)
- [Successful run 32287718197](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197)
- [vllm-ascend A3 Dockerfile](https://github.com/vllm-project/vllm-ascend/blob/367b8e62da799870a7476ce34f5f7658589a8aad/Dockerfile.a3)
- [Current FL Ascend Dockerfile](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/docker/ascend/Dockerfile)
- [FL plugin entry points](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/pyproject.toml#L50-L54)
- [`PlatformFL` worker selection](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/platform.py#L193-L220)
- [`WorkerFL` constructs `ModelRunnerFL`](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/worker/worker.py#L422-L431)
- [Ascend per-op dispatch policy](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/config/ascend.yaml)
- [`vendor.ascend` attention ownership](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/backends/vendor/ascend/ascend.py#L116-L140)
- [`vllm_fl` Ascend attention direct torch_npu calls](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/backends/vendor/ascend/impl/attention.py)

## CI image 的准确角色

| 问题 | 结论 | 状态 |
|---|---|---|
| 只是 CANN/torch-npu/Python基础环境？ | 否；还装 empty vLLM、vllm-ascend package/custom kernels、triton-ascend等 | Confirmed |
| 实际激活 vllm-ascend platform？ | 否；entry point可见但被filter，FL被激活 | Confirmed |
| CI卸载/覆盖/屏蔽package？ | 不卸载、不覆盖；只在plugin loading层过滤 | Confirmed |
| 使用 `VLLM_TARGET_DEVICE=empty`？ | 是，构建 official vLLM 0.20.2 source | Confirmed |
| 证明无package也能跑？ | 不可；没有negative control，但该问题不再是formal route的先决门禁 | Unknown / non-blocking |
| package存在是否证明vllm-ascend backend执行？ | 否；必须区分installed/discoverable与activated/imported/called | Confirmed distinction |
| 最准确描述 | 经验证的Ascend软件栈carrier；FL取得platform ownership，operator ownership预期由FL dispatch决定；动态闭包待trace | Confirmed + Unknown |

## `ascend+empty` 的含义

`empty` 是 official vLLM 的构建目标，不是 NPU backend：它只安装 common requirements，清除 vLLM device extension modules，不构建 CUDA/ROCm/CPU native backend；OOT platform plugin负责 device registration、Worker、ModelRunner和backend。它不同于普通 official CUDA wheel。Schema fallback也不等于真实NPU kernel，kernel仍来自FlagGems/Triton或torch-npu/CANN。

## Ascend 环境变量

| 变量 | 用法 | 说明 |
|---|---|---|
| `VLLM_PLUGINS` | `fl` | 只激活FL entry point |
| `VLLM_FL_PLATFORM` | `ascend` | 固定FL dispatch platform |
| `VLLM_VENDOR` | **unset** | setup只支持`cuda`；`ascend`会报错，不是platform selector |
| `TRITON_ALL_BLOCKS_PARALLEL` | R0不先设；后续单变量 | README称必需，但成功CI没设置；GLM边界Unknown |
| `FLAGCX_PATH` | baseline不设 | 默认HCCL；FlagCX后置 |

## FlagTree / Triton / FlagGems 冲突

| 来源 | 内容 |
|---|---|
| FL README | FlagGems `3b2b55c8`、FlagTree `0.4.0+ascend3.2`、eager |
| FL Ascend Docker | FlagGems `3e6528cf`；不装FlagTree |
| vllm-ascend base | `triton-ascend==3.2.1` |
| FlagGems metadata | 另有FlagTree 0.5 / Triton 3.5 / torch组合 |
| FlagTree current main | Ascend向Triton 3.5演进；安装会替换`triton` namespace |

结论为 **Conflicting**。不能拼成“最新兼容版本”，也不能把FlagTree画成vllm-ascend backend代理。Triton类实际链应追踪为`FlagGems/vendor.ascend kernel -> Triton API -> FlagTree provider或triton-ascend provider -> CANN`；当前official carrier内实际provider仍需runtime inventory/trace确认。非Triton的`vendor.ascend`/Reference可直接走`PyTorch/torch_npu -> CANN`。

FlagCX在 current CI 未安装，`PlatformFL`默认回退HCCL。FlagCX有Ascend adaptor，但不是单机/TP canary前提。

## 当前official-first路线（建议，不是安装脚本）

```text
A3/910C
  ↓ Driver/Firmware（exact待产品矩阵/现场冻结）
official FL Ascend carrier
  quay.io/ascend/vllm-ascend:v0.20.2rc1-a3
  CANN + torch/torch_npu + official vLLM empty + carrier packages/providers
  ↓ VLLM_PLUGINS=fl; VLLM_FL_PLATFORM=ascend
PlatformFL -> WorkerFL -> ModelRunnerFL
  ↓ FlagOS Dispatch per operator
FlagGems / vllm_fl vendor.ascend / NPU-resident Reference
  ↓ Triton provider -> CANN，或PyTorch/torch_npu -> CANN
Official Carrier FL-only Environment Smoke
  remove vllm-ascend only in disposable experiment container
  minimal negative check + Platform/Worker/Runner/Dispatch + synthetic NPU op
  ↓ Qwen3.6-27B TP2 eager canary
```

每层必要性：Driver/Firmware使设备可用；carrier复用official已验证的CANN/torch-npu/vLLM/编译器组合；empty vLLM提供API/engine；FL接管platform/worker/runner/dispatch；FlagGems、`vendor.ascend`与Reference按per-op policy提供实现；Triton provider只负责对应kernel编译链。FlagCX/FlagScale/ModelSlim均不默认进入第一次runtime provenance。

official carrier与静态ownership为 **Confirmed**。A2只验证卸载后的FL-only最小闭环；official coexistence的完整动态ownership仍为 **Unknown until Post-Eager Audit**。

## A2 FL-only minimal evidence

- 使用本机已有且RepoDigest/image ID明确的official carrier；缺失或digest不确定时停止，不pull；
- 仅在新建一次性实验container内卸载vllm-ascend，不修改原始image或其他carrier runtime；
- negative check只覆盖distribution不存在、`find_spec("vllm_ascend") is None`、无有效`vllm_ascend:...` entry point；
- torch/torch_npu/NPU仍可用；PlatformFL、WorkerFL、ModelRunnerFL与Dispatch可确认；
- 至少一个小shape、NPU-resident synthetic operator经FlagOS selected impl成功；
- compiler仅inventory，除非synthetic smoke自然触发，否则不扩张provider trace。

这项受控卸载只用于减少变量，不恢复“package presence即违规”的旧规则，也不能证明official coexistence路线有问题。若FL需要editable安装，必须从container内保留`.git`并通过exact HEAD/tree/clean校验的writable副本安装；正式repo保持readonly。三项negative check必须由卸载后的新Python process执行。

## Post-Eager Runtime Provenance验收证据（Deferred / On-demand）

- Eager Correctness后比较A组official coexistence与B组同carrier FL-only路线；
- image digest、distribution/module/entry-point/native-library inventory完整；其中vllm-ascend的存在本身不判FAIL；
- `current_platform`实际为`PlatformFL`，worker class实际为`WorkerFL`，runner实例实际为`ModelRunnerFL`；
- FlagOS Dispatch已注册并对关键operator记录候选顺序、最终selected impl、module/class/function origin；
- 对每个关键operator区分FlagGems、`vllm_fl...vendor.ascend`与Reference，并验证tensor/device不发生静默CPU fallback；
- 记录`sys.modules`、import audit、loaded Python/native modules和调用栈，明确是否存在任何`vllm_ascend`实际参与；
- Triton类kernel记录`triton` distribution owner、active driver/provider与CANN下游；非Triton路径记录torch_npu/CANN op；
- 若发现`vllm_ascend`实际import/call，输出具体调用、必要性与影响，暂停合规结论并交control裁定。

该审计只在客户要求coexistence证明、正式方案考虑保留vllm-ascend distribution、FL-only与coexistence出现行为差异，或最终交付需要完整provenance证据时触发；不作为Baseline Benchmark默认门禁。

## Unknown

- 客户现场Driver/Firmware/ATB exact version；
- official carrier在现场的exact digest与A2卸载后基础栈完整性；
- `vllm_ascend`是否在目标进程中有任何实际import/call/side effect，以及若有是否属于客户允许边界；
- active compiler究竟是FlagTree还是triton-ascend provider；
- `TRITON_ALL_BLOCKS_PARALLEL=1`对GLM是否必需；
- 客户是否也禁止FL内historical adapted code。
