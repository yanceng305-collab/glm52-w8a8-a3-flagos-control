# DeepSeek Prompt — A2-V024-PY311-FIRST-FLAGTREE-ENV

状态：**COMPLETED — feasibility evidence only / Do not rerun**

唯一合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-PY311-FIRST-FLAGTREE-ENV.md`

当前clean-room prompt：[`DEEPSEEK-A2-V024-CLEANROOM-CANN900-PY311-PROMPT.md`](DEEPSEEK-A2-V024-CLEANROOM-CANN900-PY311-PROMPT.md)。

## 目标

基于当前exact vLLM-Ascend v0.24 carrier创建disposable container，优先把容器运行环境切换或重建为Python3.11，并尽快闭合：

```text
Python 3.11
  -> vLLM 0.24
  -> torch / torch_npu
  -> FlagTree 0.6.1+ascend3.5
  -> FlagGems
  -> vllm-plugin-FL
  -> FlagOS Dispatch
  -> tiny NPU smoke
```

不要求保留Python3.12，也不要继续把py312兼容当作必须解决的问题。当前目标是找到能让vLLM0.24 + FlagOS在A3上工作的可重复、可维护环境。

## 固定输入

- Carrier：`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`
- Code repo：`/data/tiankuan/zyg/repos/vllm-plugin-FL-a3-flagos/`
- Control repo：`/data/tiankuan/zyg/repos/glm52-w8a8-a3-flagos-control/`
- Code base：branch=`project/glm52-w8a8-v024`，commit=`a9435a34dcd7d0a38e3a853535947371a6c62205`，tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagTree target：`0.6.1+ascend3.5`
- FlagGems baseline：v5.3.4 / commit=`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`
- Task ID：`A2-V024-PY311-FIRST-FLAGTREE-ENV`
- Run ID：UTC timestamp
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-PY311-FIRST-FLAGTREE-ENV/<run-id>/`
- Work：`/data/tiankuan/zyg/work/A2-V024-PY311-FIRST-FLAGTREE-ENV/<run-id>/`

## 执行策略

1. 同步并验证两个长期repo；不使用`/root/vllm-plugin-FL/`或legacy副本作源码。
2. 从exact carrier创建全新disposable container，记录base image ID/digest，不覆盖base image。
3. 在container内优先安装或构建Python3.11；可以替换container用户态Python和package环境，不要求保留Python3.12。
4. 在Python3.11下恢复并验证vLLM0.24、torch、torch_npu。根据实际CANN/Driver/Python ABI自行选择兼容package、wheel、source build或安装方式。
5. 安装并验证FlagTree`0.6.1+ascend3.5`，处理distribution、ABI、build、toolchain、native extension和provider overlay问题。
6. 恢复FlagGems、vllm-plugin-FL与FlagOS Dispatch，保持FL exact code identity。
7. 遇到单个package、依赖、ABI、build、import、toolchain或native-extension故障时，先自主调查、适配并继续推进，不要立即STOP。
8. 静态环境闭合后，重新检查共享资源安全；必要时仅使用NPU 12+13执行极小torch_npu tensor、FlagTree kernel和一个FlagGems/FL/Dispatch operator smoke，不执行collective。

## 排障自主权

- 可自主联网访问完成任务所需的公开/官方文档、源码、package index、registry、CI artifact和合理镜像；不采用固定网络白名单。
- 可在disposable container、work和artifacts中重装Python3.11对齐依赖、安装build/debug工具、构建wheel、生成lock/manifest/Dockerfile、创建唯一tag的derived image、比较其他FlagOS artifact/image并patch临时代码。
- 不要求使用固定命令或预选package组合。以实际兼容性和可重放结果为准。
- 记录关键命令、错误、URL、source ref、package version、wheel SHA256、image digest和适配理由；不得泄露credential。
- 不修改Host全局Python/CANN/Driver，不影响现有任务，不删除或覆盖base image和已有内容。

## Code与正式Evidence

- Exploratory阶段允许dirty working tree。
- 正式PASS/STOP前，actual execution source必须绑定exact repo、branch、commit和tree。
- 如果需要修改vllm-plugin-FL，创建`task/A2-V024-PY311-FIRST-FLAGTREE-ENV`，commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- 第三方临时patch保存在work/artifacts，不向第三方repo push。
- 所有可复用wheel、source archive、environment recipe和derived image记录SHA256/digest及重放方法。

## PASS

只有全部满足才报告PASS：

- exact carrier派生环境稳定运行Python3.11；
- vLLM0.24、torch、torch_npu可用并通过最小NPU tensor验证；
- FlagTree`0.6.1+ascend3.5`形成single coherent provider；
- 最小FlagTree kernel在Ascend成功compile/execute；
- FlagGems、FL plugin和FlagOS Dispatch smoke通过；
- tensor保持在NPU且无silent CPU fallback；
- 环境构建可重放，Code/Control/Evidence三指针完整。

## STOP

不要因单个错误立即STOP。只有合理的现场调查与适配路径确实无法继续闭合时才STOP。

STOP必须报告：

- 实际做到哪一步；
- root cause与可复现failure；
- 尝试过哪些Python、package、build、ABI、toolchain、依赖和临时patch适配；
- 哪些组件和门禁已PASS；
- 剩余blocker；
- 推荐下一路线及需要的Decision。

## 禁止

- 修改Host全局Python/CANN/Driver；
- 覆盖base image或影响现有任务；
- GLM/Qwen模型加载、vLLM serve、HCCL/TP、KV cache、完整Worker/ModelRunner runtime、benchmark/profile；
- 删除/覆盖现有Evidence、legacy、artifacts或长期repo；
- direct push integration branch、force push、把exploratory dirty运行当正式Evidence。

结束后生成immutable`results/A2-V024-PY311-FIRST-FLAGTREE-ENV/<run-id>.md`并更新`results/INDEX.md`。Control non-fast-forward允许fetch/rebase安全重试；真实冲突时Control Sync=PENDING，实验PASS/STOP不受影响。
