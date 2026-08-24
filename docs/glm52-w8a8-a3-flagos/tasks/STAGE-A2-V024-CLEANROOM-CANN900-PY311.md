# A2-V024-CLEANROOM-CANN900-PY311 — Official CANN A3 Clean-room Reproduction

状态：**Ready after user dispatch**
执行对象：第一台Ascend A3/910C服务器

## 目标

从已冻结的Ascend官方CANN 9.0.0 A3 Python3.11 devel image开始，在全新disposable container中独立安装并验证：

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

上一轮copied-runtime PY311 PASS只作feasibility evidence，不是本任务runtime来源或正式acceptance。

## Frozen base

- Image：`swr.cn-south-1.myhuaweicloud.com/ascendhub/cann@sha256:5f20011b2c5509ca4716393e66fc7aa07016629bce36a7f6c32c1bf31f30433f`
- User-confirmed source tag：`9.0.0-a3-ubuntu22.04-py3.11-devel`
- DeepSeek只需验证服务器本地image的RepoDigest、image ID、architecture、created metadata和实际Python/CANN/A3 ops/NNAL/toolchain inventory。
- Registry发现、pull排障和Dockerfile fallback build不属于本任务执行路径；历史official Dockerfile只保留为source背景。

## Identity与目录

- Task ID：`A2-V024-CLEANROOM-CANN900-PY311`
- Run ID：UTC timestamp
- Code base：`project/glm52-w8a8-v024@a9435a34dcd7d0a38e3a853535947371a6c62205`，tree`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagTree：`0.6.1+ascend3.5`
- FlagGems：v5.3.4 / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`
- Work：`/data/tiankuan/zyg/work/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`

## Clean-room边界

- 禁止使用上一轮container、runtime、work、venv、site-packages、wheelhouse、derived image或patched文件。
- 禁止从任何其他image/container复制Python、site-packages、FlagTree或vLLM。
- 禁止跨Python版本复制package。
- FlagTree是本任务正式compiler provider；不得预装或保留独立triton-ascend distribution路线，最终必须是single coherent provider。

上一轮3个问题必须在clean unmodified环境自然验证，禁止提前patch：

1. Triton`do_bench_npu`circular import；
2. FlagGems DSA缺失`__init__.py`；
3. FL`patch_mamba_config`/`cbor2`。

问题未出现时保存未复现证据；稳定复现后保存baseline repro，再调查和适配。

## 安装与Code规则

- 从零安装torch/torch_npu、vLLM0.24、FlagTree、FlagGems和FL；vLLM必须来自可解释的正式source/package路线。
- DeepSeek可自主联网调查、pip/source install、安装build/debug依赖、检查ABI/native extension、构建wheel、生成Dockerfile/script/lock/manifest、创建唯一tag derived image和临时patch third-party source。
- 所有来源记录URL、repo、commit/tag/version、SHA256/image digest、适用Python/architecture和重放方法。
- 如果需要修改vllm-plugin-FL，必须创建`task/A2-V024-CLEANROOM-CANN900-PY311`，commit、push并创建base=`project/glm52-w8a8-v024`的PR。原始FL直接通过时PR=`N/A`。
- 不得临时patch FL后仍声明Code clean/PR=N/A。
- Third-party patch必须保存patch、unmodified source identity、patched artifact SHA256和重放方法。

## 验证顺序

1. 验证本地exact digest image identity。
2. 创建全新disposable container并inventory Python/CANN/A3 ops/NNAL/toolchain。
3. 安装并验证torch + torch_npu。
4. 安装并验证vLLM0.24。
5. 安装FlagTree`0.6.1+ascend3.5`并审计provider。
6. 安装FlagGems v5.3.4。
7. 安装exact FL source并激活FlagOS Dispatch。
8. 静态闭合后重新检查共享NPU资源。
9. 仅使用NPU 12+13执行tiny torch_npu tensor、minimal FlagTree kernel和一个FlagGems/FL/Dispatch operator；禁止collective。

## PASS

只有全部满足才PASS：

- local base image精确匹配frozen digest；
- clean container从零建立且未复制上一轮/其他image runtime；
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

CANN9.0.0出现明确且无法合理适配的版本blocker，或合理调查后仍无法闭合时，STOP并报告完成阶段、root cause、repro、已尝试适配、已PASS门禁、剩余blocker、3个历史问题状态和推荐下一路线。未经Decision不得切换到CANN9.0.1。

## 非目标

- 不修改Host全局Python/CANN/Driver，不覆盖frozen base，不影响现有任务。
- 不加载模型、不运行vLLM serve、HCCL/TP、KV cache、完整Worker/ModelRunner、benchmark/profile。
- 不删除/覆盖现有Evidence、legacy、artifacts或长期repo。
- 不direct push integration branch、不force push、不把exploratory dirty或uncommitted FL patch当正式Evidence。

任务结束后生成immutable`results/A2-V024-CLEANROOM-CANN900-PY311/<run-id>.md`并更新`results/INDEX.md`。
