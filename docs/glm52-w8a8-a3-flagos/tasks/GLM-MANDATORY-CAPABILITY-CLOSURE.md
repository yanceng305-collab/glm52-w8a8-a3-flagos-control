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

| Capability | 当前证据状态 | 是否直接进入实现 |
|---|---|---:|
| Ascend MLA | Missing | 是，gap contract批准后 |
| Ascend DSA/SFA | Missing | 是，gap contract批准后 |
| Generic FL Indexer framework | Implemented | 否；只实现Ascend closure中的已确认缺口 |
| Ascend/910C Indexer closure | Missing / Unwired | 是，按backend/kernel/cache子缺口拆分 |
| Compressed-tensors W8A8 contract/packed glue | Implemented | 否；作为输入contract |
| OOT/NPU INT8 Linear kernel / 910C W8A8 runtime | Missing | 是，gap contract批准后 |
| W8A8 MoE | Implemented but unverified | 先microgate；仅失败并确认缺口后进入实现 |
| Required MoE/router/shared-expert/TP-HCCL/KV Cache | 混合状态 | 先microgate；只实现被证明阻塞首次forward的缺口 |

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

现有能力若microgate PASS，直接保留，不为了“统一实现”重写。只有状态被证据确认成Missing/Unwired才进入实现。

### 2. Minimal Capability Implementation

- 一个capability gap原则上对应一个独立task、branch和Draft PR；MLA、DSA/SFA、Indexer、W8A8不得默认打包。
- 只实现使同一microgate通过所需的最小行为；不得夹带graph、MTP、multistream、FlagCX、fusion、production tuning或无关重构。
- 生产ownership必须是：

```text
vLLM
  -> vllm-plugin-FL
  -> FlagOS dispatch/backend
  -> FlagGems / vendor.ascend
  -> torch_npu / CANN
```

- 共享接口确实不可分割时，执行者必须先提交合并理由、影响、回滚和可归因测试，由Codex书面批准后才能合并task/PR。

### 3. Corresponding Microgate PASS

实现后必须重新运行原gap contract中的同一microgate。PASS只证明：

1. backend在目标dispatch路径可达；
2. 小shape correctness满足reference/tolerance；
3. checkpoint/runtime contract、weight/scale/packing、KV/cache或owner/shared语义正确；
4. 接口、shape和dtype可被GLM-5.2 forward调用；
5. 相关既有microgate与Qwen canary无回归。

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
- 不得把vLLM-Ascend distribution/module/entry point/native runtime作为生产依赖；
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

参考策略不改变formal environment要求：

- 无vllm-ascend image；
- 无vllm-ascend distribution/package；
- 无`vllm_ascend` module；
- 无vllm-ascend entry point；
- 无运行时动态依赖；
- 不允许先安装再卸载构造环境。

当前工作解释是客户禁止runtime/package/environment dependency，不禁止开发阶段研究official vLLM-Ascend源码。只有客户以后进一步明确“official FlagOS仓库中的历史adapted来源也禁止”，才重新执行合规判断；在此之前不得自行扩大限制。

## 非目标

- 不做完整性能实现；
- 不要求graph、MTP、multistream、FlagCX、fusion或production级优化；
- 不处理First Eager Load后才出现的新整模型问题；这类问题归后续Minimal Compatibility；
- 不批准服务器、DeepSeek提示词或代码实现。

## Exit与结果合同

本Stage链只有在每个mandatory capability均有独立证据化结果时结束：

- `completed/PASS`：microgate标准全部满足；
- `partial`：保留已验证子能力，剩余gap重新拆task；
- `blocked`：缺contract、primitive、reference、权限或目标环境；
- `failed`：实现/路线不满足contract，回到gap confirmation或GLM Contract Gate。

Codex逐项验收后才能把Capacity & Placement标记Ready；执行者不得自行跨Stage。
