# A3-CP-A2 — FlagOS Runtime Provenance Trace

状态：**Proposed / Not Ready**
Parent：`Stage A — FlagOS Runtime Provenance`
执行授权：无；本文件仅定义下一条建议Stage，不是DeepSeek提示词
第二台A3：不需要

## 目标

在第一台A3/910C上，使用official FL Ascend Dockerfile对应的A3 environment carrier候选，生成可复核的运行时来源证据，证明实际执行是否为：

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

本任务回答runtime ownership，不验证GLM-5.2-W8A8 capability，不以package存在性替代动态证据。

## 已确认输入

- official FL current `main`：`92a6f7670465922c60e88f06787b8f0923e761f3`，tree `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`。
- Dockerfile base：`quay.io/ascend/vllm-ascend:v0.20.2rc1-a3`。
- Dockerfile selectors：`VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`。
- FL entry point返回`vllm_fl.platform.PlatformFL`；`PlatformFL`配置`WorkerFL`；`WorkerFL`构造`ModelRunnerFL`。
- `ascend.yaml`使用per-op顺序；`vendor.ascend` class/function位于`vllm_fl`并直接调用PyTorch/torch_npu。

以上均为static-checked或CI evidence，不是目标服务器field verification。

## Ready gate

在任何服务器/container操作前必须同时具备：

1. 用户明确批准本任务执行。
2. carrier image exact tag/digest与是否允许使用本机已有image冻结；未经批准不得pull。
3. Host device/driver/DCMI/HCCN mounts与container runtime启动边界冻结。
4. 只允许最小import、NPU-resident operator smoke和必要的短进程trace；不得启动Qwen或GLM服务。
5. Evidence目录、原始日志、敏感环境变量过滤、停止/清理规则冻结。
6. 不进行install/uninstall、Driver/CANN/网络修改、代码修改或第二台服务器访问。

## Trace面

### 1. Environment carrier

- image repo/tag/digest、OS/arch、Python、CANN、torch、torch-npu、vLLM、vllm-plugin-FL、FlagGems、Triton/FlagTree、vllm-ascend inventory；
- vLLM是否为`VLLM_TARGET_DEVICE=empty`构建的可验证证据；
- platform/general entry points及`VLLM_PLUGINS`过滤结果。

### 2. FL platform chain

- `current_platform`对象的class、module、source file；
- effective worker class及实例origin；
- effective model runner class及实例origin；
- `VLLM_FL_PLATFORM`、device type、distributed backend和eager/compile状态。

### 3. FlagOS Dispatch

- dispatch registry是否加载FlagGems、`vendor.ascend`、Reference；
- effective `ascend.yaml`及每个被测operator候选顺序；
- 代表性operator的selected impl id、callable module/file、fallback事件和failure；
- 至少覆盖attention backend、RMSNorm、SiLU，以及一条MoE或纯PyTorch/reference路径。

该覆盖只证明dispatch机制，不声称MLA/DSA/Indexer/W8A8已支持。

### 4. Compiler/provider

- installed `triton`相关distribution及import origin；
- active Triton driver/provider究竟是FlagTree还是triton-ascend；
- 对最小Triton kernel记录provider到CANN的证据；
- 对非Triton路径记录PyTorch/torch_npu到CANN的证据；
- 不允许把FlagTree画成`vllm-ascend` backend代理。

### 5. vllm-ascend participation audit

- package/module/entry point/native artifacts是否存在；
- 进程生命周期内`sys.modules`、import audit、module origin与loaded native libraries；
- 如发现`vllm_ascend`实际import/call，记录触发者、call site、输入职责、是否位于关键operator路径、是否绕过FlagOS Dispatch、下游和可替换边界；
- 如未发现，只能在已覆盖进程/算子范围内写“not observed”，不得外推为全栈不存在。

## Exit / 验收

| Gate | PASS条件 |
|---|---|
| Platform | runtime class明确为`PlatformFL` |
| Worker | runtime class明确为`WorkerFL` |
| ModelRunner | runtime class明确为`ModelRunnerFL` |
| Dispatch | 代表性operator经过FlagOS registry/policy并记录selected impl |
| Backend ownership | selected callable属于FlagGems、`vllm_fl...vendor.ascend`或Reference，origin可复核 |
| Device | tensor与执行留在NPU，无静默CPU fallback |
| Compiler | Triton路径的active provider已识别；非Triton路径与torch_npu/CANN已区分 |
| vllm-ascend | installed与runtime participation分开报告；任何实际调用都有精确分类 |
| Scope | 不加载模型、不改环境、不进入GLM capability实现 |

任一关键项证据不足写`Unknown`，来源冲突写`Conflict`，可能阻断official route写`Potential Blocker`。不得为了PASS隐藏carrier package或动态调用。

## 结果对后续路线的影响

- 若FL chain、Dispatch和下游均闭合，且未观察到`vllm_ascend`实际参与：接受official carrier作为下一步Qwen canary候选。
- 若观察到`vllm_ascend`参与：暂停对该调用的合规结论，由Codex按ownership、必要性和客户边界决定接受、隔离、替换或设计对照实验。
- 若FL chain未激活或关键operator绕过Dispatch：Stage失败，回到environment/plugin activation设计，不进入Qwen或GLM。
- 若compiler provider无法判定：Stage partial，不进入依赖Triton结论的capability gate。

## 非目标

- 不操作第二台服务器；
- 不启动Qwen或GLM模型；
- 不验证MLA、DSA/SFA、Indexer、W8A8；
- 不安装、卸载或替换vllm-ascend；
- 不构建neutral CANN对照container；
- 不生成DeepSeek执行提示词；
- 不进行benchmark、profile或性能优化。

## Static evidence

- [official Ascend Dockerfile](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/docker/ascend/Dockerfile)
- [FL entry points](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/pyproject.toml#L50-L54)
- [`vllm_fl.register`](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/__init__.py#L105-L126)
- [`PlatformFL` worker selection](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/platform.py#L193-L220)
- [`WorkerFL` runner construction](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/worker/worker.py#L422-L431)
- [Ascend per-op policy](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/config/ascend.yaml)
- [`vendor.ascend` attention selector](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/backends/vendor/ascend/ascend.py#L116-L140)
- [`vllm_fl` attention direct torch_npu implementation](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/vllm_fl/dispatch/backends/vendor/ascend/impl/attention.py)
