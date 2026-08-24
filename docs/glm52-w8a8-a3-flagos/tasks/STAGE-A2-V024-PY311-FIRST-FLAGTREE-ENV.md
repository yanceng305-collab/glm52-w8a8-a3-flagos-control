# A2-V024-PY311-FIRST-FLAGTREE-ENV — Maintainable FlagOS A3 Environment

状态：**COMPLETED — feasibility evidence only / formal clean-room validation required / Do not rerun**
执行对象：第一台Ascend A3/910C服务器

Immutable result：`../results/A2-V024-PY311-FIRST-FLAGTREE-ENV/20260824T065902Z.md`。当前task：[`STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`](STAGE-A2-V024-CLEANROOM-CANN900-PY311.md)。

## 目标

基于exact vLLM-Ascend v0.24 carrier创建disposable container，优先把容器用户态运行环境切换或重建为Python 3.11，并闭合：

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

Python 3.12不再是必须保留或解决的目标。项目优先获得能在A3上稳定工作、可重复构建和可维护的vLLM0.24 + FlagOS环境。

## 固定边界

- Base carrier：`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`
- Code base：`project/glm52-w8a8-v024@a9435a34dcd7d0a38e3a853535947371a6c62205`，tree`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- Target FlagTree：`0.6.1+ascend3.5`
- FlagGems project baseline：v5.3.4 / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`
- Host长期repo：`/data/tiankuan/zyg/repos/`
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-PY311-FIRST-FLAGTREE-ENV/<run-id>/`
- Work：`/data/tiankuan/zyg/work/A2-V024-PY311-FIRST-FLAGTREE-ENV/<run-id>/`

不得修改Host全局Python、CANN、Driver，不覆盖base image，不影响现有任务。所有runtime替换、依赖重装、构建与patch只发生在disposable container、work或artifacts中。

## 执行自主权

- DeepSeek可自主选择Python3.11安装/切换方式、torch/torch_npu兼容组合、vLLM0.24重装方式、build toolchain、package source和debug手段。
- 可访问排障所需的公开/官方文档、源码仓库、package index、container registry、CI artifact和合理镜像；不使用固定网络白名单。
- 可在disposable container/work/artifacts内重装与Python3.11对齐的依赖、安装build/debug工具、构建wheel、生成lock/manifest/Dockerfile、创建唯一tag的derived image、比较其他artifact/image并patch临时代码。
- 每个单独的ABI、package、build、toolchain、dependency、import或native-extension错误都应先调查和适配，不得因单个问题立即STOP。
- 外部source/artifact必须记录URL、version/ref、SHA256、适用Python/architecture和实际用途；不得泄露credential。

## Code与provenance

- Exploratory阶段允许dirty working tree；正式PASS/STOP必须绑定actual execution source的exact repo、branch、commit和tree。
- 如果修改vllm-plugin-FL，创建`task/A2-V024-PY311-FIRST-FLAGTREE-ENV`，commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- 临时FlagTree/third-party patch保存在work/artifacts；未经单独授权不向第三方repo push。
- 任何可复用wheel、source archive、environment recipe或derived image都必须记录hash/digest与重放方法。

## 执行路线

1. 从exact carrier创建全新disposable container，冻结base image与Host边界。
2. 在container内安装或构建Python3.11 runtime；不要求保留Python3.12 environment。
3. 为Python3.11恢复可用的vLLM0.24、torch和torch_npu，并验证与container CANN/Host Driver兼容。
4. 安装并验证FlagTree`0.6.1+ascend3.5`，形成single coherent provider。
5. 安装/验证FlagGems v5.3.4与FL exact code identity，激活FlagOS Dispatch。
6. 处理过程中遇到的依赖、ABI、build、import和native-extension问题，保存repro与适配结果并继续推进。
7. 环境静态闭合后，重新确认共享资源安全；必要时仅使用NPU 12+13执行极小torch_npu tensor、FlagTree kernel及一个FlagGems/FL/Dispatch operator smoke。

## PASS

只有以下全部成立才报告PASS：

- exact carrier派生的disposable环境稳定运行Python3.11；
- vLLM0.24、torch、torch_npu身份明确并完成最小NPU device/tensor验证；
- FlagTree`0.6.1+ascend3.5`distribution、RECORD、module、native extension、backend origin一致，无混合provider；
- 最小FlagTree kernel在Ascend成功compile/execute；
- FlagGems、FL plugin与FlagOS Dispatch smoke通过，tensor留在NPU且无silent CPU fallback；
- environment recipe、依赖清单、wheel/hash或derived image digest足以重放；
- Code、Control、Evidence三指针完整。

## STOP

只有在合理的现场调查与适配路径确实无法继续闭合时才STOP。STOP result必须明确：

- 实际完成到哪一步；
- 最小root cause及可复现failure；
- 尝试过的安装、构建、patch、依赖和toolchain适配；
- 已PASS组件/门禁；
- 剩余blocker；
- 推荐下一路线及需要的Decision。

## 非目标与安全

- 不加载GLM/Qwen，不启动vLLM serve，不执行HCCL/TP、KV cache、完整Worker/ModelRunner runtime、benchmark/profile。
- 不修改Host全局runtime，不覆盖base image，不删除或覆盖现有Evidence、legacy、artifacts或长期repo。
- 不direct push integration branch，不force push，不把source-affecting dirty运行当正式Evidence。

任务结束后生成immutable`results/A2-V024-PY311-FIRST-FLAGTREE-ENV/<run-id>.md`并更新`results/INDEX.md`。Control同步失败按现行规则处理，不改变实验PASS/STOP。
