# A2-V024-CLEANROOM-CANN900-PY311 — Official CANN A3 Clean-room Reproduction

状态：**Ready after user dispatch**
执行对象：第一台Ascend A3/910C服务器

## 目标

从全新的Ascend官方CANN 9.0.0 A3 Python3.11 devel基础环境开始，独立安装并验证：

```text
Python 3.11
  -> CANN 9.0.0 A3
  -> torch / torch_npu 2.10.x
  -> vLLM 0.24
  -> FlagTree 0.6.1+ascend3.5
  -> FlagGems v5.3.4
  -> vllm-plugin-FL project/glm52-w8a8-v024
  -> FlagOS Dispatch
  -> tiny NPU smoke
```

上一轮`A2-V024-PY311-FIRST-FLAGTREE-ENV/20260824T065902Z`只作feasibility evidence，不是正式环境acceptance，也不是本任务的runtime来源。

## Official clean base

优先使用用户已在AscendHub确认存在的官方tag：

`9.0.0-a3-ubuntu22.04-py3.11-devel`

- DeepSeek必须从AscendHub官方页面或官方pull说明取得实际完整image ref；Control不猜registry地址。
- Pull后冻结完整image ref、RepoDigest、image ID、architecture、created metadata和实际OS/Python/CANN/toolkit/ops/NNAL inventory。
- 如果官方tag经合理排障仍无法取得，可从Ascend官方repo exact source本地build同一CANN 9.0.0基线，不得切换到9.0.1：
  - Repo：`https://github.com/Ascend/cann-container-image`
  - Commit：`aec636189a2304f704871460e9fcaa0b7f22dde3`
  - Dockerfile：`cann/9.0.0-a3-ubuntu22.04-py3.11/Dockerfile`
  - Dockerfile Git blob：`e1abd27dfa72e0c7e6a8f666253acb585a02af99`
- Local build必须从clean exact commit执行，并冻结derived image digest、build command、source tree及下载artifact provenance。

Official Dockerfile source contract为Ubuntu22.04、Python3.11.15、CANN9.0.0 toolkit、A3 ops与NNAL；实际环境仍以pull/build后的inventory为准。

## Identity与目录

- Task ID：`A2-V024-CLEANROOM-CANN900-PY311`
- Run ID：UTC timestamp
- Code base：`project/glm52-w8a8-v024@a9435a34dcd7d0a38e3a853535947371a6c62205`，tree`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagTree：`0.6.1+ascend3.5`
- FlagGems：v5.3.4 / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`
- Work：`/data/tiankuan/zyg/work/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`

## 首选兼容组合

第一优先参考FlagGems当前`ascend-cann900`组合：

- Python 3.11
- torch 2.10.0+cpu
- torch-npu 2.10.0
- triton 3.5.0
- triton-ascend 3.2.1
- FlagTree`0.6.1+ascend3.5`

这是首选实验基线，不是死限制。若CANN/Driver/vLLM0.24存在冲突，DeepSeek先调查并合理适配。最终FlagTree必须是single coherent provider，不得留下FlagTree/triton-ascend混杂文件或双重ownership。

## 强制clean-room边界

- 禁止从此前FlagOS Qwen image复制Python、site-packages、FlagTree、vLLM、torch/torch_npu、FlagGems、FL或其他runtime。
- 禁止从任何其他container直接复制site-packages或FlagTree。
- 禁止跨Python版本复制vLLM package。
- 禁止复用上一轮container、work runtime、venv、site-packages、wheelhouse或derived image。
- 只允许读取上一轮immutable result/Evidence理解历史问题，禁止复制其runtime和patched文件。

上一轮3处临时patch不得提前应用：

1. Triton`do_bench_npu`circular-import patch；
2. FlagGems DSA缺失`__init__.py`patch；
3. FL`patch_mamba_config`/`cbor2`patch。

这些问题必须在clean unmodified source/package中自然重新验证。未出现则保存未复现证据；稳定复现则保存baseline repro后再作为真实适配缺口处理。

## 安装与排障原则

- torch/torch_npu、vLLM0.24、FlagTree、FlagGems和FL必须从可解释的正式source/package路线独立安装。
- vLLM0.24不得从py312 site-packages或其他container复制。
- DeepSeek拥有现场排障与合理联网自主权，可使用官方/可信GitHub、GitCode、AscendHub、package index、CI artifact、文档和镜像。
- 可在clean disposable container/work/artifacts中pip/source install、安装build/debug依赖、检查ABI/native extension、构建wheel、生成Dockerfile/script/lock/manifest、创建唯一tag derived image和临时patch third-party source。
- 所有source/artifact必须记录URL、repo、commit/tag/version、SHA256/image digest、适用Python/architecture及重放方法。
- 单个dependency、ABI、build、toolchain、import或native-extension问题先调查适配，不立即STOP。

## Code与Patch规则

- 如果需要修改vllm-plugin-FL，必须创建`task/A2-V024-CLEANROOM-CANN900-PY311`，commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- 不得临时patch FL后仍声明Code clean或PR=N/A；正式smoke必须绑定actual task commit/tree。
- FlagTree/FlagGems/Triton third-party patch可保存在work/artifacts，但必须保存patch文件、unmodified source identity、patched artifact SHA256及重放方法。
- Exploratory dirty允许；正式PASS/STOP必须绑定actual execution code和全部patched dependency identity。

## 验证顺序

1. 获取并冻结官方CANN 9.0.0 A3 py3.11-devel image identity。
2. 创建全新container，inventory Python/CANN/toolchain。
3. 安装并验证torch + torch_npu。
4. 从正式source/package安装并验证vLLM0.24。
5. 安装FlagTree`0.6.1+ascend3.5`并审计single-provider ownership。
6. 安装FlagGems v5.3.4。
7. 安装exact FL source并激活FlagOS Dispatch。
8. 静态闭合后重新检查共享NPU资源。
9. 仅使用NPU 12+13执行tiny torch_npu tensor、minimal FlagTree kernel和一个FlagGems/FL/Dispatch operator；禁止collective。

## PASS

只有全部满足才PASS：

- 官方CANN 9.0.0 A3 py3.11 devel base identity明确；
- clean container从零构建，不依赖上一轮runtime复制；
- torch/torch_npu在NPU可运行；
- vLLM0.24 source/package identity明确；
- FlagTree`0.6.1+ascend3.5`single coherent provider，无混杂残留；
- FlagTree kernel在Ascend compile + execute PASS；
- FlagGems + FL + Dispatch smoke PASS；
- tensor保持NPU且无silent CPU fallback；
- 上一轮3个问题均有clean baseline重现/未重现证据；
- 所有必要patch具有正式provenance，FL patch已进入task branch/commit/PR；
- 环境有可重放Dockerfile/script/manifest/lock或derived image digest；
- actual execution code identity与Git指针一致；
- Code、Control、Evidence三指针完整。

## STOP

如果CANN 9.0.0本身出现明确且无法合理适配的版本blocker，或合理调查后仍无法闭合，STOP并报告完成阶段、root cause、repro、尝试过的适配、已PASS门禁、剩余blocker、patch状态和推荐下一路线。未经Decision不得切换到CANN9.0.1。

## 非目标与安全

- 不修改Host全局Python/CANN/Driver，不覆盖official base，不影响现有任务。
- 不加载GLM/Qwen，不serve，不HCCL/TP，不KV cache，不运行完整Worker/ModelRunner，不benchmark/profile。
- 不删除/覆盖现有Evidence、legacy、artifacts或长期repo。
- 不direct push integration branch、不force push、不把exploratory dirty或uncommitted FL patch当正式Evidence。

任务结束后生成immutable`results/A2-V024-CLEANROOM-CANN900-PY311/<run-id>.md`并更新`results/INDEX.md`。
