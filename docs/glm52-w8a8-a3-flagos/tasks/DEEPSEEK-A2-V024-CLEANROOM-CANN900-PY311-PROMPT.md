# DeepSeek Prompt — A2-V024-CLEANROOM-CANN900-PY311

状态：**Ready after user dispatch**

唯一合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`

## 目标

从已冻结的Ascend官方CANN 9.0.0 A3 Python3.11 devel image开始，在全新disposable container中从零建立：

```text
Python 3.11 / CANN 9.0.0 A3
  -> torch / torch_npu 2.10.x
  -> vLLM 0.24
  -> FlagTree 0.6.1+ascend3.5
  -> FlagGems v5.3.4
  -> vllm-plugin-FL project/glm52-w8a8-v024
  -> FlagOS Dispatch
  -> tiny NPU smoke
```

上一轮PY311 PASS只作feasibility evidence，不得复用其runtime、container、work或patched文件。

## Frozen base

- Image：
  `swr.cn-south-1.myhuaweicloud.com/ascendhub/cann@sha256:5f20011b2c5509ca4716393e66fc7aa07016629bce36a7f6c32c1bf31f30433f`
- User-confirmed source tag：`9.0.0-a3-ubuntu22.04-py3.11-devel`

不要再查询pull地址、排障registry或执行Dockerfile fallback build。本任务首先验证服务器本地image精确匹配该digest，并记录image ID、architecture、created metadata及实际Python/CANN/A3 ops/NNAL/toolchain inventory。

## Task identity

- Task ID：`A2-V024-CLEANROOM-CANN900-PY311`
- Run ID：UTC timestamp
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`
- Work：`/data/tiankuan/zyg/work/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`
- Code repo：`/data/tiankuan/zyg/repos/vllm-plugin-FL-a3-flagos/`
- Control repo：`/data/tiankuan/zyg/repos/glm52-w8a8-a3-flagos-control/`
- Code base：branch=`project/glm52-w8a8-v024`，commit=`a9435a34dcd7d0a38e3a853535947371a6c62205`，tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagTree：`0.6.1+ascend3.5`
- FlagGems：v5.3.4 / commit=`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`

## 强制clean-room边界

严禁：

- 使用上一轮container、runtime、work、venv、site-packages、wheelhouse、derived image或patched文件；
- 从任何其他image/container复制Python、site-packages、FlagTree或vLLM；
- 跨Python版本复制package；
- 提前应用上一轮patch。

FlagTree是本任务正式compiler provider。不要预装独立triton-ascend路线；最终不得存在FlagTree/triton-ascend混杂文件、dist-info或双重ownership。

上一轮3个问题必须在clean unmodified环境自然验证：

1. Triton`do_bench_npu`circular import；
2. FlagGems DSA缺失`__init__.py`；
3. FL`patch_mamba_config`/`cbor2`。

问题未出现时保存未复现证据；稳定复现后保存baseline repro，再自主调查和适配。

## 执行顺序

1. 验证本地exact digest image identity。
2. 从该digest创建全新disposable container。
3. Inventory Python3.11、CANN9.0.0、A3 ops、NNAL和toolchain。
4. 从零安装并验证torch + torch_npu。
5. 从可解释的正式source/package路线安装vLLM0.24；禁止跨Python或跨container复制。
6. 安装FlagTree`0.6.1+ascend3.5`并审计single-provider ownership。
7. 安装FlagGems v5.3.4。
8. 安装exact vllm-plugin-FL。
9. 激活FlagOS Dispatch。
10. 静态闭合后重新检查共享NPU资源。
11. 仅使用NPU 12+13执行：
    - tiny torch_npu tensor；
    - minimal FlagTree kernel；
    - 一个FlagGems/FL/Dispatch operator。

禁止collective。

## 排障自主权

- 可自主使用官方/可信网络、GitHub、GitCode、package index、CI artifact、文档和合理镜像。
- 可在clean container/work/artifacts中pip/source install、安装build/debug依赖、检查ABI/native extension、构建wheel、生成Dockerfile/script/lock/manifest、创建唯一tag derived image和临时patch third-party source。
- 单个dependency、ABI、build、toolchain、import或native-extension问题先调查适配，不立即STOP。
- 所有来源记录URL、repo、commit/tag/version、SHA256/image digest、适用Python/architecture和重放方法。

## Code与Patch规则

- 如果需要修改vllm-plugin-FL，创建`task/A2-V024-CLEANROOM-CANN900-PY311`。
- FL修改必须commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- 正式smoke必须绑定该task branch exact commit/tree。
- 原始FL直接通过时PR=`N/A`。
- 不能临时patch FL后仍声明Code clean或PR=N/A。
- FlagTree/FlagGems/Triton patch必须保存patch文件、unmodified source identity、patched artifact SHA256和重放方法。
- Exploratory dirty允许；正式PASS/STOP必须绑定actual execution code和全部patched dependency identity。

## PASS

只有全部满足才PASS：

- local base image精确匹配frozen digest；
- clean container从零建立，未使用上一轮/其他image runtime复制；
- torch/torch_npu在NPU可运行；
- vLLM0.24身份明确；
- FlagTree`0.6.1+ascend3.5`single coherent provider，无triton-ascend混杂；
- FlagTree kernel在Ascend compile + execute PASS；
- FlagGems + FL + Dispatch smoke PASS；
- tensor保持NPU且无silent CPU fallback；
- 3个历史问题均有clean baseline重现/未重现证据；
- 必要patch具有正式provenance；若实际需要修改vllm-plugin-FL，则FL patch必须进入task branch/commit/PR；若原始exact FL无需修改即可通过，则PR=N/A，同样满足PASS；
- 环境有可重放Dockerfile/script/manifest/lock或derived image digest；
- actual execution code identity与Git指针一致；
- Code、Control、Evidence三指针完整。

## STOP

CANN9.0.0出现明确且无法合理适配的版本blocker，或合理调查后仍无法闭合时，STOP并报告：

- 实际做到哪一步；
- root cause与repro；
- 尝试过的依赖、build、toolchain和patch；
- 已PASS门禁；
- 剩余blocker；
- 3个历史问题状态；
- 推荐下一路线和需要的Decision。

未经Decision不得切换到CANN9.0.1。

## 禁止

- 修改Host全局Python/CANN/Driver；
- 覆盖frozen base或影响现有任务；
- 模型加载、vLLM serve、HCCL/TP、KV cache、完整Worker/ModelRunner、benchmark/profile；
- 删除/覆盖现有Evidence、legacy、artifacts或长期repo；
- direct push integration branch、force push；
- 把exploratory dirty或uncommitted FL patch当正式Evidence。

结束后生成immutable`results/A2-V024-CLEANROOM-CANN900-PY311/<run-id>.md`并更新`results/INDEX.md`。Control non-fast-forward允许fetch/rebase安全重试；真实冲突时Control Sync=PENDING，实验PASS/STOP不受影响。
