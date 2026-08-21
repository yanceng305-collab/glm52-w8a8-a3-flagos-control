# FlagOS A3/910C 环境调查与证据链

调查日期：2026-08-21
固定 FL：`38e7dbc20197e2db742c4e4c9687d36ea4df9900`

## 总判定

- **Confirmed：** 官方 910C CI image 是完整 vllm-ascend runtime，不是只提供 CANN/torch-npu 的中性底座。
- **Confirmed：** FL Docker/CI setup 不卸载、覆盖或移除 vllm-ascend；package/entry point仍 installed/discoverable。
- **Confirmed：** `VLLM_PLUGINS=fl` 过滤 platform entry points，公开 CI log 显示实际激活 `Platform plugin fl`；Worker/ModelRunner来自FL。
- **Unknown：** 官方没有从零且从未安装 vllm-ascend 的 910C negative-control job，不能证明 clean-room import/native closure。
- **Inferred：** 客户合规第一候选应从 neutral CANN A3 base 零起，复刻 CI tuple但不安装 vllm-ascend，命名 `R0-clean`。

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
910C: vllm-ascend discoverable, FL activated, Qwen3.6 TP2 success
```

关键证据：

- [FL Ascend Dockerfile](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/docker/ascend/Dockerfile)
- [FL Ascend setup](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/.github/scripts/ascend/setup.sh)
- [FL Ascend CI config](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/.github/configs/ascend.yml)
- [FL 910C matrix](https://github.com/flagos-ai/vllm-plugin-FL/blob/38e7dbc20197e2db742c4e4c9687d36ea4df9900/tests/platforms/ascend.yaml)
- [Successful run 32287718197](https://github.com/flagos-ai/vllm-plugin-FL/actions/runs/32287718197)
- [vllm-ascend A3 Dockerfile](https://github.com/vllm-project/vllm-ascend/blob/367b8e62da799870a7476ce34f5f7658589a8aad/Dockerfile.a3)

## CI image 的准确角色

| 问题 | 结论 | 状态 |
|---|---|---|
| 只是 CANN/torch-npu/Python基础环境？ | 否；还装 empty vLLM、vllm-ascend package/custom kernels、triton-ascend等 | Confirmed |
| 实际激活 vllm-ascend platform？ | 否；entry point可见但被filter，FL被激活 | Confirmed |
| CI卸载/覆盖/屏蔽package？ | 不卸载、不覆盖；只在plugin loading层过滤 | Confirmed |
| 使用 `VLLM_TARGET_DEVICE=empty`？ | 是，构建 official vLLM 0.20.2 source | Confirmed |
| 证明无package也能跑？ | 不可；没有negative control | Unknown |
| 最准确描述 | 经验证的完整Ascend软件栈载体；FL取得platform ownership，但环境仍污染 | Confirmed + Inferred解释 |

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

结论为 **Conflicting**。不能拼成“最新兼容版本”。`R0-clean` 为保持单变量先用 actual CI 的 triton-ascend 3.2.1 且不装 FlagTree；`R1-compiler` 再独立验证 FlagTree，严禁叠装。

FlagCX在 current CI 未安装，`PlatformFL`默认回退HCCL。FlagCX有Ascend adaptor，但不是单机/TP canary前提。

## 客户合规路线（建议，不是安装脚本）

```text
A3/910C
  ↓ Driver/Firmware（exact待产品矩阵/现场冻结）
Neutral CANN layer: Ubuntu 22.04 + CANN 9.0.0 A3 + matching ATB/NNAL
  ↓ Python 3.11.15
torch 2.10.0 + torch-npu 2.10.0
  ↓ single Triton provider: triton-ascend 3.2.1 (R0)
official vLLM 0.20.2 empty source build
  ↓ FlagGems 3e6528cf without conflicting extras
vllm-plugin-FL approved current-main SHA, Python-only
  ↓ HCCL baseline
Clean Provenance tests
  ↓ Qwen3.6-27B TP2 eager canary
```

每层必要性：Driver/Firmware使设备可用；CANN/ATB/HCCL提供NPU runtime/native ops/collectives；torch-npu提供PyTorch NPU接口；empty vLLM提供API/engine；FL接管platform/worker/runner/dispatch；FlagGems提供统一operator；Triton provider编译kernel。FlagTree/FlagCX/ModelSlim都不默认进入第一baseline。

该路线在910C canary通过前只能标 **Inferred**。

## Clean Provenance 负证据

- `pip show vllm-ascend`失败；`find_spec("vllm_ascend")`为空；无其entry point/native library/import trace；
- 从fresh base生成，从未安装后卸载；
- `current_platform`唯一为`PlatformFL`；
- official vLLM是empty source build；
- package/native/import inventory与环境hash齐全；
- torch-npu device、HCCL、FlagGems/Triton kernel、FL最小test通过；
- fresh rebuild结果一致。

## Unknown

- 客户现场Driver/Firmware/ATB exact version；
- neutral CANN image是否允许；只允许host install时，公开FL无完整recipe；
- 去掉vllm-ascend后是否缺隐式transitive/native artifact；
- FlagTree最终兼容profile；
- `TRITON_ALL_BLOCKS_PARALLEL=1`对GLM是否必需；
- 客户是否也禁止FL内historical adapted code。
