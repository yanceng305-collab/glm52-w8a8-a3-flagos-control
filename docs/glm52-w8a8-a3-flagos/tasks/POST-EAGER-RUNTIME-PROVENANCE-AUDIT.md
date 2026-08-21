# Post-Eager Runtime Provenance Audit

状态：**Deferred / Not Ready**
Stage位置：GLM-5.2-W8A8 `Eager Correctness`之后、`Baseline Benchmark`之前
前身：`A3-CP-A2 — FlagOS Runtime Provenance Trace`；原trace设计保留于本合同并后置
第二台A3：默认不需要

## 工程意图

在目标GLM-5.2-W8A8 eager correctness已经闭合后，对official carrier coexistence与FL-only环境做受控对照，回答完整runtime ownership问题。该审计不阻塞首次bring-up，也不把vllm-ascend image/package存在性当成违规。

对照profile：

```text
A. official carrier保留vllm-ascend distribution + VLLM_PLUGINS=fl
B. 同一carrier的一次性实验container内卸载vllm-ascend，保留同一FL/model/config
```

两组必须尽可能保持image digest、FL/vLLM、checkpoint、并行布局、输入与eager配置一致。差异只用于识别package coexistence下的动态参与，不用于性能宣称。

## Ready gate

1. A3-CP-A2 FL-only Environment Smoke已PASS。
2. Qwen canary、GLM Contract、mandatory capability closure、First Eager Load、Minimal Compatibility与Eager Correctness已按PLAN验收。
3. 真实GLM-5.2-W8A8 checkpoint、exact runtime配置和golden/tolerance证据冻结。
4. A/B对照container、image digest、device范围、Evidence和停止条件获得单独执行批准。
5. 当前NPU资源足以串行执行，不影响其他任务；默认不使用第二台服务器。

## 审计目标链

```text
vLLM
  -> vllm-plugin-FL entry point
  -> PlatformFL
  -> WorkerFL
  -> ModelRunnerFL
  -> FlagOS Dispatch
  -> FlagGems / vllm_fl vendor.ascend / NPU-resident Reference
  -> Triton API -> FlagTree或triton-ascend provider -> CANN
     OR PyTorch/torch_npu -> CANN
  -> Ascend 910C
```

`vllm-ascend` distribution可能存在于A组carrier，但不能预先画成runtime hop；其动态参与必须由本Stage证据决定。

## Trace面

### 1. Environment与entry points

- A/B image、container、Python、CANN、torch、torch-npu、vLLM、FL、FlagGems、Triton/FlagTree、vllm-ascend inventory；
- vLLM empty-build证据；
- platform/general entry points、`VLLM_PLUGINS`过滤与plugin activation；
- A/B差异manifest。

### 2. FL platform chain

- `current_platform`对象的class/module/source file；
- effective worker class与实例origin；
- effective model runner class与实例origin；
- device type、distributed backend、eager/compile状态；
- A/B中上述来源是否一致。

### 3. FlagOS Dispatch与关键operator

- dispatch registry及effective `ascend.yaml`；
- 目标模型关键operator的候选顺序、selected impl id、callable module/file、fallback与failure事件；
- 分别记录FlagGems、`vllm_fl...vendor.ascend`、Reference路径；
- 确认tensor/device不发生静默CPU fallback；
- 核查是否有关键执行绕过FlagOS Dispatch。

覆盖范围至少由已通过的GLM mandatory closure决定，包括W8A8 Linear/MoE、MLA、DSA/SFA、Indexer及首次eager实际触达的必要runtime；不得用synthetic smoke代替目标模型路径。

### 4. Compiler/provider完整ownership

- installed Triton相关distribution与import origin；
- active Triton driver/provider究竟是FlagTree还是triton-ascend；
- 对Triton kernel记录provider、codegen/runtime与CANN下游；
- 对非Triton路径记录PyTorch/torch_npu/CANN调用；
- FlagTree只作为compiler/provider，不得解释成vllm-ascend backend代理。

### 5. vllm-ascend dynamic participation

- A组中distribution/module/entry point/native artifacts的存在与来源；
- 进程生命周期`sys.modules`、import audit、module origin、loaded native libraries和必要调用栈；
- 任何`vllm_ascend` import/call的触发者、call site、职责、输入输出、是否位于关键operator路径、是否绕过FlagOS Dispatch及下游；
- B组对应路径是否消失或改变；
- “not observed”只覆盖实际trace到的进程、阶段与operator，不外推全栈不存在。

### 6. Source provenance与runtime dependency分离

- `vllm_fl`内`Adapted from vllm-ascend`文件继续按source provenance、license与attribution记录；
- 只有动态import/call/link evidence才用于判断runtime dependency；
- 不得把历史adapted来源自动归类为vllm-ascend runtime参与。

## Exit / 验收

| Gate | PASS条件 |
|---|---|
| A/B comparability | 除vllm-ascend coexistence变量外，关键runtime/model配置可比 |
| Platform chain | 两组PlatformFL/WorkerFL/ModelRunnerFL来源明确 |
| Dispatch | GLM关键operator均有selected backend与module/function origin |
| Device | 关键tensor/执行保持NPU，无静默CPU fallback |
| Compiler | active provider、Triton与非Triton下游完整可追溯 |
| vllm-ascend | A组任何动态参与均被精确分类；B组差异可解释 |
| Ownership | 是否存在绕过FlagOS Dispatch的关键执行有明确结论 |
| Scope | 不把审计结果冒充性能或新correctness验收 |

结果必须区分Confirmed / Unknown / Conflict / Potential Blocker。若发现`vllm_ascend`实际参与，不自动判违规；由Codex对具体调用的ownership、必要性、客户边界和替换成本做后续决策。

## 必存证据

- A/B immutable image/container/runtime inventory与差异manifest；
- class/module/function/entry-point origin；
- FlagOS dispatch policy和per-op selected backend trace；
- `sys.modules`、import audit、loaded Python/native library、必要调用栈；
- Triton active provider与torch_npu/CANN trace；
- GLM关键operator覆盖矩阵；
- commands、stdout/stderr、timestamps、checksums；
- 限定适用边界的审计结论。

## 非目标

- 不重新证明首次eager correctness；
- 不把package presence作为PASS/FAIL；
- 不做benchmark/profile或以A/B差异宣称性能结论；
- 不自动修改、替换或删除runtime实现；
- 不扩大客户限制到official FL历史adapted源码。

## Static evidence

- [official Ascend Dockerfile](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/docker/ascend/Dockerfile)
- [FL entry points](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/pyproject.toml#L50-L54)
- [`vllm_fl.register`](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/__init__.py#L105-L126)
- [`PlatformFL` worker selection](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/platform.py#L193-L220)
- [`WorkerFL` runner construction](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/worker/worker.py#L422-L431)
- [Ascend per-op policy](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/config/ascend.yaml)
- [`vendor.ascend` attention selector](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/backends/vendor/ascend/ascend.py#L116-L140)
- [`vllm_fl` attention direct torch_npu implementation](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/backends/vendor/ascend/impl/attention.py)
