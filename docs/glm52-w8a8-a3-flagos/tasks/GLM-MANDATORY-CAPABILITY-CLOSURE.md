# GLM Mandatory Capability Closure Stage Contract

状态：Proposed；not ready；当前不授权实现或服务器执行

## 目标与项目贡献

在Capacity & Placement和First Eager Load之前，把第一次正确GLM-5.2-W8A8 eager token所需的mandatory能力按同一闭环治理：

```text
Capability Microgates / Gap Confirmation
  -> Minimal Capability Implementation
  -> Corresponding Microgate PASS
  -> Capacity & Placement
```

本合同同时定义vLLM-Ascend的技术参考边界，避免把“允许研究参考实现”误解为“允许复制源码或引入runtime依赖”。

## 适用能力

| Capability | 当前证据状态 | Gap Confirmation处理 |
|---|---|---:|
| Ascend MLA | 静态评估Missing | 重新审查FlagGems/vendor.ascend/Reference；三路失败才进入实现 |
| Ascend DSA/SFA | 静态评估Missing | 重新审查A/B/C路径；三路失败才进入实现 |
| Generic FL Indexer framework | Implemented | 审查可达backend；不得因缺FlagGems自动判Missing |
| Ascend/910C Indexer closure | Missing / Unwired | 按backend/kernel/cache子缺口执行A/B/C路径审查 |
| Compressed-tensors W8A8 contract/packed glue | Implemented | 作为输入contract；审查现有执行路径 |
| OOT/NPU INT8 Linear kernel / 910C W8A8 runtime | 静态评估Missing | 必须审查FlagGems、vendor.ascend及NPU-resident Reference |
| W8A8 MoE | Implemented but unverified | 先microgate；仅A/B/C均失败并确认缺口后实现 |
| Required MoE/router/shared-expert/TP-HCCL/KV Cache | 混合状态 | 先microgate；只实现三路均不可用且阻塞首次forward的缺口 |

任何非mandatory能力、纯性能问题或未来组合优化不得进入本Stage。

## 统一工作循环

### 1. Capability Microgates / Gap Confirmation

Codex先形成capability gap contract，至少包含：

- FlagOS当前缺口与可达路径；
- official vLLM/Transformers模型contract；
- 输入输出、tensor shape、dtype、layout；
- KV Cache、prefill/decode、owner/shared语义（适用时）；
- Ascend需要的torch_npu/CANN primitive与910C约束；
- 当前FlagOS ownership/dispatch落点；
- 最小correctness reference、tolerance、failure signature；
- 是否确属首次eager mandatory closure。

每个operator/capability必须按以下顺序审查当前FlagOS可用路径：

1. FlagGems；
2. `vendor.ascend`；
3. Reference/PyTorch通用实现。

任一路径只有同时满足以下条件才判PASS：在FlagOS Dispatch内可达；在910C执行；满足microgate correctness；接口支撑GLM-5.2 forward。Reference tensor必须留在NPU并通过torch_npu/CANN执行；必须记录tensor device、selected dispatch/backend和必要的profiler/import trace，发现静默CPU fallback即失败。vllm-ascend image/package存在只作environment inventory；如该capability trace发现`vllm_ascend`实际import/call，必须准确归因并交control判断，不得自动判PASS或FAIL。

现有能力若任一A/B/C路径microgate PASS，直接保留，不为了统一来源或FlagGems覆盖率重写。只有三条路径全部失败，状态才可确认成Missing/Unwired并进入实现。

### 2. Minimal Capability Implementation

- 一个capability gap原则上对应一个独立task、branch和Draft PR；MLA、DSA/SFA、Indexer、W8A8不得默认打包。
- 只实现使同一microgate通过所需的最小行为；不得夹带graph、MTP、multistream、FlagCX、FlagScale、fusion、production tuning、FlagGems覆盖率开发或无关重构。
- 生产ownership必须是：

```text
vLLM
  -> vllm-plugin-FL
  -> FlagOS dispatch/backend
  -> FlagGems / vendor.ascend / Reference
  -> torch_npu / CANN
```

- 共享接口确实不可分割时，执行者必须先提交合并理由、影响、回滚和可归因测试，由Codex书面批准后才能合并task/PR。

### 3. Corresponding Microgate PASS

实现后必须重新运行原gap contract中的同一microgate。PASS只证明：

1. backend在目标dispatch路径可达；
2. 小shape correctness满足reference/tolerance；
3. checkpoint/runtime contract、weight/scale/packing、KV/cache或owner/shared语义正确；
4. 接口、shape和dtype可被GLM-5.2 forward调用；
5. device/backend trace证明tensor与执行留在NPU，无静默CPU fallback；
6. 相关既有microgate与Qwen canary无回归。

全部mandatory capability获得PASS证据后，Capacity & Placement才可变成Ready。

## vLLM-Ascend技术参考规则（嵌入实现循环）

### 允许研究和提取

Codex可深入阅读official vLLM-Ascend并提取：

- 输入输出contract；
- tensor shape、dtype和layout；
- KV Cache组织；
- owner/shared语义；
- prefill/decode行为；
- torch_npu/CANN primitive选择；
- 910C特殊约束；
- correctness/reference行为。

这些内容必须先转写成与源码文本解耦的implementation contract，再交给未来实现任务。

### 参考优先级

1. FlagOS已有跨平台实现；
2. official vLLM / Transformers模型contract；
3. official vLLM-Ascend作为Ascend/910C hardware implementation reference。

较低优先级reference不能覆盖较高优先级的模型语义或FlagOS ownership；冲突必须在contract中显式记录。

### 禁止的实现方式

- 不得直接复制vLLM-Ascend文件后通过变量名、类名、目录或机械重构隐藏来源；
- 不得让新增capability绕过FlagOS Dispatch而直接以vLLM-Ascend backend作为未经审查的生产owner；环境carrier/package存在不属于本条禁止范围；
- 不得用代码混淆规避license、copyright或attribution；
- 不得因为reference已有实现就跳过FlagOS dispatch设计、correctness test或microgate。

如果确实复用或派生源码，必须在PR中明确文件/commit/范围，遵守对应许可证与attribution要求；不满足时不得合入。

### 每个capability patch的必填说明

- FlagOS当前缺口；
- vLLM/Transformers模型contract；
- Ascend hardware primitive；
- FlagOS ownership/dispatch路径；
- correctness test与未覆盖边界；
- vLLM-Ascend作为技术reference的具体范围；
- 是否存在源码复用/派生、相应license与attribution；
- 回滚方式与Qwen/其他mandatory microgate回归结果。

## 正式运行环境边界

本节已按runtime ownership修正；此前“image/distribution/module/entry point必须全部不存在、环境必须从未安装”的presence-based规则已**Superseded**。

- official同款vllm-ascend A3 image可作为环境carrier；
- distribution/module/entry point的installed/discoverable状态必须如实记录，但存在性本身不判违规；
- 实际模型执行必须由`PlatformFL -> WorkerFL -> ModelRunnerFL -> FlagOS Dispatch`拥有；
- 每个capability必须记录最终FlagGems、`vendor.ascend`或Reference owner及torch_npu/CANN下游；
- 发现任何`vllm_ascend`动态import/call时，记录call site、职责、是否绕过Dispatch和必要性，由control单独判断客户边界/替换；
- 不以先安装再卸载来制造缺席证据，也不在未trace时声称完全独立。

允许的bring-up执行链为：

```text
FlagOS Dispatch
  -> FlagGems / vendor.ascend / Reference
  -> torch_npu / CANN
  -> Ascend 910C
```

当前工作解释是客户核心要求为FlagOS runtime/dispatch/backend ownership，不禁止official carrier/package存在，也不禁止开发阶段研究official vLLM-Ascend源码。只有runtime trace发现具体参与调用，或客户以后进一步明确“official FlagOS仓库中的历史adapted来源也禁止”，才对对应范围重新执行合规判断；在此之前不得自行扩大限制。

## 非目标

- 不做完整性能实现；
- 不要求graph、MTP、multistream、FlagCX、fusion或production级优化；
- FlagScale不作为本Stage或First Eager Load前置，只能在模型链稳定后的后期集成阶段验证；
- 不处理First Eager Load后才出现的新整模型问题；这类问题归后续Minimal Compatibility；
- 不批准服务器、DeepSeek提示词或代码实现。

## Eager Correctness后的性能重评估

Reference路径通过correctness并不代表最终性能接受。只有Eager Correctness通过且profiling证明其为瓶颈后，才分别比较FlagGems、vendor.ascend、Triton/FlagTree和kernel fusion；每种优化仍按单变量、独立正确性回归和Draft PR治理。

## Exit与结果合同

本Stage链只有在每个mandatory capability均有独立证据化结果时结束：

- `completed/PASS`：microgate标准全部满足；
- `partial`：保留已验证子能力，剩余gap重新拆task；
- `blocked`：缺contract、primitive、reference、权限或目标环境；
- `failed`：实现/路线不满足contract，回到gap confirmation或GLM Contract Gate。

Codex逐项验收后才能把Capacity & Placement标记Ready；执行者不得自行跨Stage。
