# Historical A2 Contract — FlagOS Runtime Provenance Trace

状态：**Superseded as pre-canary A2 / Preserved and moved post-eager**

本文件原定义`A3-CP-A2 — FlagOS Runtime Provenance Trace`。其动态import/call、loaded native library、per-op ownership、compiler/provider和vllm-ascend coexistence审计设计没有删除，已完整保留并后置为：

- [`POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`](POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md)

当前前置A2改为：

- [`STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)

这是工程优先级调整，不是恢复“vllm-ascend image/package存在即违规”的旧规则。前置A2在一次性实验container内卸载vllm-ascend，只用于减少首次bring-up变量；official carrier中保留package并通过`VLLM_PLUGINS=fl`运行仍是合法、待Post-Eager审计的路线。

本历史文件不得作为DeepSeek执行任务下发。
