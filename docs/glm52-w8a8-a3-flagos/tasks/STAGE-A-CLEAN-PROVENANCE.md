# Stage A Task Contract — Clean Provenance

状态：Proposed；不是 ready
当前禁止执行者：DeepSeek / server operator（等待用户后续授权）

## 目标

在一台 A3/910C 上，从客户允许的中性 Ascend/CANN 基础层开始，建立一个从未安装过 vllm-ascend image/package/runtime/plugin 的 `R0-clean`，证明最小 NPU、HCCL、Triton provider、FlagGems 和 FL platform 链路闭合。

本 Stage 不加载 Qwen/GLM，不实现 GLM patch，不测试性能，不引入 FlagTree、FlagCX 或 ModelSlim。

## Ready 条件

1. 用户明确授权使用一台 A3 server；本轮尚未授权。
2. 用户确认允许中性 CANN image，或指定只允许 host clean install。
3. Driver/Firmware/CANN 的产品兼容依据和只读 inventory 权限齐备。
4. 新 repository/控制面已确认并成为 GitHub 唯一事实源。
5. 客户确认“原生”是否允许官方 FL 仓库内 historical adapted code。
6. 固定任务时重新读取 official FL current main；不得自动沿用调查 SHA。

## 环境假设（待实验，不是安装脚本）

`R0-clean` 第一候选：Ubuntu 22.04 userland、Python 3.11.15、CANN 9.0.0 A3、torch/torch-npu 2.10.0、triton-ascend 3.2.1、official vLLM 0.20.2 empty source build、FlagGems `3e6528cf`、FL current-approved SHA；`VLLM_VENDOR` unset，`VLLM_PLUGINS=fl`，`VLLM_FL_PLATFORM=ascend`。

Driver/Firmware/ATB exact version保持 Unknown，必须由产品矩阵/现场 inventory 冻结，不能从本假设补写。

## Exit / 验收

1. `vllm-ascend` distribution不存在；`find_spec("vllm_ascend")`为空；不存在其 platform entry point、native library 或 import trace。
2. official vLLM 确认为 `VLLM_TARGET_DEVICE=empty` source build，无 device-native vLLM extension。
3. 唯一激活 OOT platform 是 FL；`current_platform=PlatformFL`、device=`npu`；`VLLM_VENDOR` unset。
4. Driver/Firmware/CANN/Python/torch/torch-npu/Triton/FlagGems/FL inventory机器可读；未知字段明确 Unknown。
5. torch-npu device smoke、HCCL collective、代表性 FlagGems/Triton kernel、FL 最小 unit/functional 集通过。
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
- 执行 owner（未来授权后）：DeepSeek。
- 验收 owner：Codex；进入下一 Stage 需要用户批准。
