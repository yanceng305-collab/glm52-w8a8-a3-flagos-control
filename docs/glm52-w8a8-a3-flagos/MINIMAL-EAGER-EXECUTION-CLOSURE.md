# Minimal Eager Execution Closure

状态：Proposed；closure unchanged；primary baseline refreshed to FlagOS new main / vLLM0.24；未执行服务器或实现

目标：定义得到第一个**正确的 GLM-5.2-W8A8 eager token**所需的最小能力闭环。该闭环只服务首次目标模型forward/eager correctness，不提前追求完整性能实现。

## 结论

**Confirmed within the frozen evidence set（GLM-5.2 config `b4734de`、Transformers `e0e7504`、vLLM 0.23.0 `0fc695f`、vLLM0.24 `ee0da84`、FL new-main ancestor `bb439d`）：当前不存在官方支持、代码可达且有correctness依据的Dense MLA fallback；GLM-5.2第一次eager bring-up不能绕过DSA与Indexer。**

因此第一次目标模型closure固定为：

```text
GLM-5.2-W8A8 real checkpoint
  + vLLM 0.24 GLM-5.2 semantics / IndexShare behavior
  + MLA
  + DSA semantics / reachable sparse-attention backend (SFA or equivalent)
  + Indexer full/shared ownership and Ascend backend/kernel closure
  + W8A8 Linear + W8A8 MoE runtime
  + required router/shared-expert/TP-HCCL/model-loader runtime
  + eager
  -> first correct token
```

W8A8、MLA、DSA和Indexer均为首次目标模型closure的硬门禁。MTP可以关闭；graph、multistream、FlagCX和fusion优化不属于该闭包。

## 为什么DSA与Indexer不能后移

### 1. 官方GLM-5.2 config固定启用DSA/Indexer

官方checkpoint config包含`model_type=glm_moe_dsa`、`index_topk=2048`、`index_topk_freq=4`、逐层`indexer_types`和`index_share_for_mtp_iteration=true`，[见固定config](https://huggingface.co/zai-org/GLM-5.2/blob/b4734de4facf877f85769a911abafc5283eab3d9/config.json#L20-L26)。这些不是推理端可选的性能参数，而是目标模型结构/执行语义。

### 2. Transformers eager仍是sparse DSA，不是Dense fallback

Transformers v5.12.0 config把每层`layer_types`设为`deepseek_sparse_attention`，[configuration](https://github.com/huggingface/transformers/blob/e0e7504bca2bfd1b85bb0eedb148f7b250226f06/src/transformers/models/glm_moe_dsa/configuration_glm_moe_dsa.py#L122-L157)。模型代码中：

- full层构造`GlmMoeDsaIndexer`，shared层复用上一full层top-k；
- eager/SDPA路径把top-k转为additive sparse mask；
- shared层没有先前top-k会直接报错。

证据：[GlmMoeDsaAttention与Indexer owner/shared](https://github.com/huggingface/transformers/blob/e0e7504bca2bfd1b85bb0eedb148f7b250226f06/src/transformers/models/glm_moe_dsa/modeling_glm_moe_dsa.py#L347-L470)。因此“使用eager attention implementation”只改变kernel/interface，不会把模型改成Dense Attention。

### 3. vLLM 0.23与current primary 0.24都把`index_topk`绑定到sparse MLA/Indexer

在官方最低支持版本vLLM 0.23.0：

- `hasattr(config, "index_topk")`使GLM路径进入v3.2/GLM sparse分支；
- 创建Indexer并把`is_sparse=True`交给MLA wrapper；
- `index_topk_freq/offset`只决定owner/shared复用，不能关闭DSA。

证据：[vLLM 0.23 GLM/DSA构造](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py#L999-L1074)。

Current primary vLLM0.24保留同一MLA/DSA/Indexer结构，并由FL [`bb439d...`](https://github.com/flagos-ai/vllm-plugin-FL/commit/bb439d028479475a965712e08ce0b955fe02aafb)在NVIDIA TP16双机完成GLM-5.2-Slim初始化与weight loading后进入MLA cache gap。该PARTIAL证据强化了mandatory closure，但不证明Ascend/W8A8实现。

### 4. `VLLM_MLA_DISABLE`不是合法的GLM-5.2 Dense MLA fallback

vLLM把该变量描述为“disable MLA attention optimizations”，[env定义](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/envs.py#L1281-L1282)。设置后`model_config.use_mla=False`并选择`DeepseekV2Attention`，[selection](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py#L1115-L1130)；但GLM-5.2仍因`index_topk`分配top-k buffer，[allocation](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py#L1232-L1241)，而`DeepseekV2Attention`要求该buffer必须为`None`，[assertion](https://github.com/vllm-project/vllm/blob/0fc695fc6d1d82e9a5ac6835ac8e4e1c83703665/vllm/model_executor/models/deepseek_v2.py#L409-L445)。

即使通过非官方patch删除`index_topk/indexer_types`或绕过断言，执行也会从top-k稀疏mask变成full attention，改变官方Transformers和checkpoint定义的模型语义；当前没有官方correctness证据允许把它作为首次目标模型验收。

## 最小必须能力

| 能力 | 首次eager是否必须 | 最小验收 |
|---|---:|---|
| 真实GLM-5.2-W8A8 checkpoint contract | 是 | manifest、quant metadata、tensor/scale layout由runtime正确识别 |
| W8A8 Linear | 是 | OOT/NPU INT8 kernel可达，小规模结果对reference正确，目标linears可forward |
| W8A8 MoE | 是 | routed/shared expert scale与计算正确；router按checkpoint保留精度执行 |
| MLA | 是 | Ascend backend可达，prefill/decode最小shape正确 |
| DSA semantics / SFA或等价backend | 是 | top-k sparse attention可达；小规模结果与官方eager/reference语义一致 |
| Indexer framework | 是 | full层计算top-k，shared层复用正确 |
| Ascend Indexer kernel closure | 是 | projection/cache/logits/top-k owner明确且backend可达 |
| IndexShare checkpoint ownership | 是 | owner/shared层缺失/存在权重符合真实manifest，加载无静默错配 |
| MoE/router/shared expert runtime | 是 | 必要router、dispatch、expert、combine链可forward |
| TP/HCCL | 取决于capacity布局；大模型预计需要并行 | 最终首次布局所需collective正确 |
| eager execution | 是 | 禁用graph相关变量后目标forward产生首个正确token |
| BF16 full-model bring-up | 否，且不能替代 | 只允许operator/reference/debug microtest |
| MTP | 否 | 首次closure关闭 |
| graph/multistream/FlagCX/fusion | 否 | 后续阶段 |

## Microgate验收边界

每个mandatory microgate只要求：

1. backend可达，实际选择到目标Ascend/910C实现；
2. 小规模correctness通过，并保留reference、dtype、shape和tolerance；
3. checkpoint/runtime contract正确，权重、scale、packing和owner/shared语义不静默偏移；
4. 接口与shape足以被GLM-5.2 forward调用。

不要求graph、MTP、multistream、FlagCX、fusion优化或production级性能。

## 首次eager correctness验收

- 必须使用本项目真实GLM-5.2-W8A8 checkpoint；
- 必须走W8A8 Linear/MoE实际runtime，不能用BF16替代；
- 必须保留MLA + DSA + Indexer官方语义；
- MTP、graph、multistream、FlagCX和非必要fusion保持关闭；
- 输出首个token并与批准的reference/tolerance比较，同时保存backend trace、weight/scale audit和首轮内存记录。

本结论不批准Stage A或Capability Microgates执行，只冻结其未来验收范围。
