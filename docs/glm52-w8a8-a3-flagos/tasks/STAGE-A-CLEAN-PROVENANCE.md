# Stage A Task Contract — Clean Provenance

状态：Parent Stage Proposed；Container tuple static-resolved；环境构建仍Not Ready
当前执行边界：本轮不允许DeepSeek或server操作；等待用户批准后续最小container实验任务

## 目标

在一台A3/910C上，从批准的neutral CANN Container base开始，建立一个从未安装过vllm-ascend image/package/runtime/plugin的`R0-clean`，证明NPU、HCCL、compiler、FL platform与至少一条合法FlagOS Dispatch算子路径闭合。

本 Stage 不加载 Qwen/GLM，不实现 GLM patch，不测试性能，不引入 FlagTree、FlagCX 或 ModelSlim。

## 子任务顺序

```text
Container boundary and tuple static resolution (Complete)
  -> local image digest + minimal negative-audit experiment design (Not Ready)
  -> clean environment build task (Not Ready)
```

A3-CP-A1保留为历史/补证合同，不再是当前tuple决策前置。

## Ready 条件

1. 用户批准local image digest与minimal negative-audit实验。
2. 本机candidate RepoDigest与批准的ARM64 manifest关系明确。
3. Host Driver/Firmware/container runtime/device/network挂载合同冻结。
4. Primary Container tuple及failure-triggered fallback保持control-approved。
5. 用户另行批准environment build任务。

## 环境假设（待实验，不是安装脚本）

`R0-primary`：`R0-P1-CANN901-PY312`。Base为本机candidate`quay.io/ascend/cann:9.0.1-a3-ubuntu22.04-py3.12`，Container内使用CANN901配套Toolkit/ops/NNAL、Python3.12.13、torch2.10.0、torch-npu2.10.0.post2、triton-ascend3.2.1、official vLLM0.20.2 empty、FL`92a6f767...`、FlagGems`3e6528cf...`；FlagCX/FlagScale不前置，vllm-ascend从未安装。

条件fallback：`R0-F1-CANN900-PY311`，仅在primary失败且归因到CANN901/Python312/post2组合时使用。CANN8.5不是fallback。完整tuple和Unknown见[`../R0-CONTAINER-TUPLE-RESOLUTION.md`](../R0-CONTAINER-TUPLE-RESOLUTION.md)。

Host CANN8.5/9.0.1不参与选择；R0原则上不得bind-mount Host Toolkit。

## Exit / 验收

1. `vllm-ascend` distribution不存在；`find_spec("vllm_ascend")`为空；不存在其 platform entry point、native library 或 import trace。
2. official vLLM 确认为 `VLLM_TARGET_DEVICE=empty` source build，无 device-native vLLM extension。
3. 唯一激活 OOT platform 是 FL；`current_platform=PlatformFL`、device=`npu`；`VLLM_VENDOR` unset。
4. Driver/Firmware/CANN/Python/torch/torch-npu/Triton/FlagGems/FL inventory机器可读；未知字段明确 Unknown。
5. torch-npu device smoke、HCCL collective、代表性FlagGems/vendor.ascend/NPU-resident Reference合法路径和FL最小unit/functional集通过。
6. 从 fresh base 重建一次并得到一致 inventory 与结果；不得通过卸载 vllm-ascend实现。
7. 验收结论只覆盖 clean provenance，不冒充模型或 GLM 支持。

## 必存证据

- Host/runtime/container inventory；
- packages、entry points、modules、native libraries、`sys.modules` / import trace；
- environment definition 与 immutable hash；
- commands、stdout/stderr、test reports；
- fresh rebuild 对比；
- failure signature、偏差、Unknown、rollback/stop condition。

## 停止条件

- 基础层已含或隐式拉入 vllm-ascend；
- 需要通过“先安装后卸载”继续；
- Driver/Firmware/CANN compatibility无法确认；
- compiler provider发生冲突或同时拥有 FlagTree/triton-ascend；
- 任务需要修改 GLM 源码、控制面或进入下一 Stage。

## 资源

- 一台 A3/910C；第二台不需要。
- Container研究owner：Codex；环境构建owner仍未授权。
- 验收 owner：Codex；进入下一 Stage 需要用户批准。
