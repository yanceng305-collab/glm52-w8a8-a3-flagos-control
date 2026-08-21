# Stage A Parent Contract — Official Carrier FL-only Bring-up

状态：Parent Stage Proposed；A3-CP-A2 Prepared / Not Ready
当前执行边界：本轮不允许DeepSeek或server操作；等待用户另行批准A2执行

## 历史与当前边界

- `c70aa4b`的neutral-base-only与package-absence合规门禁继续保持Superseded。
- `78b6eb06`确认的official carrier、FlagOS runtime ownership、`vendor.ascend`归属和FlagTree provider位置继续有效。
- 原pre-canary完整Runtime Provenance Trace范围过重，已后置到Eager Correctness之后；其内容保存在[`POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`](POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md)。
- 当前A2只在一次性实验container中卸载vllm-ascend，用于减少FL-only bring-up变量；不代表official coexistence路线非法。

## Stage A目标

用本机已有、identity明确的official A3 carrier创建隔离实验container，在不加载任何模型的条件下证明：

```text
official A3 carrier
  -> remove vllm-ascend plugin in disposable container only
  -> vLLM + vllm-plugin-FL
  -> PlatformFL -> WorkerFL -> ModelRunnerFL
  -> FlagOS Dispatch
  -> FlagGems / vllm_fl vendor.ascend / NPU-resident Reference
  -> torch_npu / CANN -> Ascend 910C
```

只要求至少一个小shape synthetic operator成功；不要求完整attention、MoE、compiler或dynamic provenance。

## 子任务顺序

```text
Static official ownership review (Complete)
  -> A3-CP-A1 historical/read-only host inventory (Dormant / Not Ready)
  -> A3-CP-A2 Official Carrier FL-only Environment Smoke (Prepared / Not Ready)
  -> 910C Qwen canary (future, separate approval)
  -> GLM contract and mandatory capability closure
  -> GLM-5.2-W8A8 Eager Correctness

Deferred / on-demand side branch after Eager Correctness:
  -> Post-Eager Runtime Provenance Audit
```

## Parent Ready gate

1. 用户明确批准A3-CP-A2服务器执行。
2. 本机已有carrier RepoDigest/image ID可确认；缺失或不确定时只报告，不pull。
3. 重新检查OC2及其他任务占用，只选择明确空闲的最小logical device范围。
4. WORKDIR与唯一Evidence目录规则冻结。
5. 允许的唯一package变更是实验container内卸载vllm-ascend，以及必要时从container内部完整、保留`.git`且通过SHA/tree/clean校验的writable staging副本执行冻结FL baseline的`--no-deps` editable安装；正式repo只读。
6. 不修改Host、原始image、正式代码repo或其他carrier runtime package。

## Parent Exit

Stage A在A3-CP-A2满足以下结果时结束：

- official carrier从本机exact image启动；
- container内vllm-ascend卸载完成，三项minimal negative check由卸载后的新Python process执行并通过；
- 若安装FL，container staging的HEAD/tree/clean校验通过，生成artifact未回写正式repo；
- torch/torch-npu/NPU仍可用；
- PlatformFL、WorkerFL、ModelRunnerFL与FlagOS Dispatch确认；
- 至少一个NPU-resident synthetic operator经FlagOS ownership成功；
- Evidence与安全边界完整。

详细task contract：[`STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)。

## Stage A不覆盖

- Qwen或GLM模型权重/服务；
- MLA、DSA/SFA、Indexer、W8A8、完整MoE或attention；
- 全operator provenance、coexistence dynamic audit、native library trace；
- 深度FlagTree/triton-ascend provider识别；
- benchmark/profile/性能优化；
- 第二台服务器。

## 资源与停止

- 仅第一台A3，且只用明确空闲device；任何不确定即停止。
- 不kill任务、不修改Driver/Firmware/CANN/network、不pull/build image、不删除已有image/container。
- A2结束后立即停止，等待Codex验收；不得自行进入Qwen或GLM Stage。
