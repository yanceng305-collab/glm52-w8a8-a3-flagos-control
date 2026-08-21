# Stage A Task Contract — FlagOS Runtime Provenance

状态：Parent Stage Proposed；runtime-ownership route corrected；execution Not Ready
当前执行边界：本轮不允许DeepSeek或server操作；等待用户另行批准具体runtime trace任务

## Supersession

本父合同原先要求从neutral CANN base构建“从未安装vllm-ascend”的`R0-clean`。该presence-based门禁源自control commit `c70aa4b83b4d270ee1d920e807296d0b283cfab2`的错误前提，现已**Superseded**。历史commit与兼容性研究不删除；R0-P1/R0-F1只保留reference evidence。

当前Stage A的目标改为证明official FlagOS Ascend实际路线的运行时ownership。vllm-ascend image/distribution/module/entry point是否存在只作inventory；不得单凭存在判FAIL，也不得单凭静态无direct import判PASS。

## 目标

在一台A3/910C上，以official FL Ascend Dockerfile的A3 carrier路线为优先候选，证明：

```text
vLLM
  -> vllm-plugin-FL entry point
  -> PlatformFL
  -> WorkerFL
  -> ModelRunnerFL
  -> FlagOS Dispatch
  -> FlagGems / vllm_fl vendor.ascend / NPU-resident Reference
  -> Triton provider -> CANN，或PyTorch/torch_npu -> CANN
  -> Ascend 910C
```

本Stage不加载GLM、不实现GLM patch、不测试性能，不引入FlagCX/FlagScale作为前置，也不预先替换或卸载vllm-ascend package。

## 子任务顺序

```text
Static official ownership review (Complete)
  -> A3-CP-A1 historical/read-only host inventory (Dormant / Not Ready)
  -> A3-CP-A2 FlagOS Runtime Provenance Trace (Proposed / Not Ready)
  -> 910C Qwen canary (future, requires separate approval)
```

## Ready条件

1. 用户批准A3-CP-A2及其最小container/runtime操作边界。
2. official carrier exact image reference/digest与允许的Host device/driver/network mounts冻结。
3. trace方法能同时覆盖Python entry point/class origin、FlagOS per-op dispatch、module/import/native-library、compiler provider和torch_npu/CANN下游。
4. raw evidence目录、停止条件、敏感信息过滤和结果报告格式冻结。
5. 明确本任务不卸载package、不修改Driver/CANN/网络、不启动GLM、不进入capability实现。

## Exit / 验收

1. `VLLM_PLUGINS=fl`与`VLLM_FL_PLATFORM=ascend`有效，唯一实际platform class为`vllm_fl.platform.PlatformFL`。
2. 实际worker class为`vllm_fl.worker.worker.WorkerFL`，实际runner实例为`vllm_fl.worker.model_runner.ModelRunnerFL`。
3. FlagOS Dispatch已注册；对代表性attention、RMSNorm、SiLU/MoE及后续mandatory capability记录候选顺序、selected implementation和module/function origin。
4. FlagGems、`vendor.ascend`、Reference三类路径分开记录；Reference tensor留在NPU，无静默CPU fallback。
5. Triton类实现记录`triton` distribution owner、active driver/provider（FlagTree或triton-ascend）与CANN下游；非Triton实现记录PyTorch/torch_npu/CANN调用。
6. 完整记录vllm-ascend distribution/module/entry point/native artifacts的存在状态，并通过import/call/library trace回答目标进程是否实际使用`vllm_ascend`。
7. 若存在实际调用，报告精确call site、职责、是否绕过FlagOS Dispatch、必要性与替换影响；本Stage不自行给客户合规结论。
8. 结论只覆盖runtime provenance，不冒充Qwen/GLM correctness、W8A8或性能支持。

## 必存证据

- Host/runtime/container与immutable image inventory；
- Python distributions、entry points、module/class/function origins；
- `current_platform`、worker、runner实例证据；
- FlagOS registry/policy与per-op selected backend trace；
- `sys.modules`、import audit、loaded Python/native modules和必要调用栈；
- Triton active provider/driver与torch_npu/CANN trace；
- commands、stdout/stderr、timestamps、environment hash；
- Confirmed / Unknown / Conflict / Potential Blocker报告。

## 停止条件

- 无法区分carrier package存在与实际runtime participation；
- trace需要修改Driver/CANN/网络、卸载package、启动GLM或编写capability代码；
- compiler/provider身份仍只能由package名猜测而无动态证据；
- 发现`vllm_ascend`实际调用但无法冻结调用点/作用；
- 任务越过当前服务器、container或证据写入授权。

## 资源

- 一台A3/910C；第二台不需要。
- 研究与验收owner：Codex；服务器执行owner尚未获本轮授权。
- 下一任务见[`STAGE-A2-FLAGOS-RUNTIME-PROVENANCE-TRACE.md`](STAGE-A2-FLAGOS-RUNTIME-PROVENANCE-TRACE.md)，当前Proposed / Not Ready。
