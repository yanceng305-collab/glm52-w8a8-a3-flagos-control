# DeepSeek Prompt — A2-V024-CLEANROOM-CANN900-PY311

状态：**Ready after user dispatch**

唯一合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`

## 目标

从全新的Ascend官方CANN 9.0.0 A3 Python3.11 devel基础环境开始，完全独立安装并验证：

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

上一轮PY311 PASS只作feasibility evidence，不得复用其runtime、container、work或patched文件，也不代表本任务已通过。

## Task identity

- Task ID：`A2-V024-CLEANROOM-CANN900-PY311`
- Run ID：UTC timestamp
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`
- Work：`/data/tiankuan/zyg/work/A2-V024-CLEANROOM-CANN900-PY311/<run-id>/`
- Code repo：`/data/tiankuan/zyg/repos/vllm-plugin-FL-a3-flagos/`
- Control repo：`/data/tiankuan/zyg/repos/glm52-w8a8-a3-flagos-control/`
- Code base：branch=`project/glm52-w8a8-v024`，commit=`a9435a34dcd7d0a38e3a853535947371a6c62205`，tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`

## Official clean base

用户已在AscendHub确认官方tag存在：

`9.0.0-a3-ubuntu22.04-py3.11-devel`

不要猜registry地址。首先从AscendHub官方页面、官方pull按钮或官方说明取得实际完整image ref。

Pull后记录：

- 完整image ref；
- RepoDigest；
- image ID；
- architecture；
- created metadata；
- OS、Python、CANN toolkit、A3 ops、NNAL和toolchain inventory。

如果官方image经合理registry/network排障仍无法取得，可从官方repo固定源码本地build同一9.0.0基线，不得切换到9.0.1：

- Repo：`https://github.com/Ascend/cann-container-image`
- Commit：`aec636189a2304f704871460e9fcaa0b7f22dde3`
- Dockerfile：`cann/9.0.0-a3-ubuntu22.04-py3.11/Dockerfile`
- Dockerfile Git blob：`e1abd27dfa72e0c7e6a8f666253acb585a02af99`

Local build必须从clean exact checkout执行，记录build command、source tree、downloaded artifact provenance和derived image digest。

该Dockerfile source contract为Ubuntu22.04、Python3.11.15、CANN9.0.0 toolkit、A3 ops和NNAL；实际内容以pull/build后inventory为准。

## 首选实验组合

第一优先参考FlagGems当前`ascend-cann900`组合：

- Python 3.11
- torch 2.10.0+cpu
- torch-npu 2.10.0
- triton 3.5.0
- triton-ascend 3.2.1
- FlagTree`0.6.1+ascend3.5`

这是首选基线，不是死限制。如果CANN/Driver/vLLM0.24存在冲突，先调查并合理适配，不立即STOP。

最终FlagTree必须形成single coherent provider，不得存在FlagTree/triton-ascend混杂残留或双重ownership。

## 强制clean-room边界

本次禁止：

- 从之前FlagOS Qwen image复制Python；
- 从任何其他container复制site-packages；
- 从其他image直接复制FlagTree；
- 跨Python版本复制vLLM package；
- 复用上一轮container；
- 复用上一轮已经修改的work runtime、venv、wheelhouse或derived image；
- 复制上一轮patch后文件。

只允许读取上一轮immutable result/Evidence理解历史问题。

上一轮3处patch必须自然重新验证，禁止提前应用：

1. Triton`do_bench_npu`circular import；
2. FlagGems DSA缺失`__init__.py`；
3. FL`patch_mamba_config`/`cbor2`。

先使用unmodified exact source/package：

- 问题未出现：保存未复现证据；
- 问题稳定复现：保存完整baseline repro，再作为真实适配缺口处理。

## 安装顺序

1. 拉取并冻结官方CANN 9.0.0 A3 py3.11-devel base。
2. 创建全新container并inventory Python/CANN/toolchain。
3. 独立安装、验证torch + torch_npu。
4. 从可解释的正式source/package路线安装vLLM0.24；禁止复制py312 site-packages。
5. 安装FlagTree`0.6.1+ascend3.5`并审计provider。
6. 安装FlagGems v5.3.4 / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`。
7. 安装vllm-plugin-FL exact source。
8. 激活FlagOS Dispatch。
9. 静态闭合后重新检查共享NPU资源。
10. 仅使用NPU 12+13执行：
    - tiny torch_npu tensor；
    - minimal FlagTree kernel；
    - 一个FlagGems/FL/Dispatch operator。

禁止collective。

## 排障自主权

- 可自主使用官方/可信网络检索、GitHub、GitCode、AscendHub、package index、CI artifact、文档和合理镜像。
- 可在clean disposable container/work/artifacts中pip/source install、安装build/debug依赖、检查ABI/native extension、构建wheel、生成Dockerfile/script/lock/manifest、创建唯一tag derived image和临时patch third-party source。
- 单个dependency、ABI、build、toolchain、import或native-extension故障先调查适配，不立即STOP。
- 所有来源记录URL、repo、commit/tag/version、SHA256/image digest、适用Python/architecture和重放方法。

## Patch与Code规则

- 如果需要修改vllm-plugin-FL，创建`task/A2-V024-CLEANROOM-CANN900-PY311`。
- FL修改必须commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- 正式smoke必须绑定该task branch exact commit/tree。
- 不能临时patch FL后仍声明Code clean或PR=N/A。
- FlagTree/FlagGems/Triton third-party patch必须保存patch文件、unmodified source identity、patched artifact SHA256和重放方法。
- Exploratory dirty允许；正式PASS/STOP必须绑定actual execution code和全部patched dependency identity。

## PASS

只有全部满足才PASS：

- 官方CANN 9.0.0 A3 py3.11 devel base identity明确；
- clean container从零构建，不依赖上一轮runtime复制；
- torch/torch_npu在NPU可运行；
- vLLM0.24 source/package identity明确；
- FlagTree`0.6.1+ascend3.5`single coherent provider，无混杂残留；
- FlagTree kernel在Ascend compile + execute PASS；
- FlagGems + FL + Dispatch smoke PASS；
- tensor保持NPU，无silent CPU fallback；
- 3个历史问题均有clean baseline重现/未重现证据；
- 所有必要patch具有正式provenance，FL patch已进入task branch/commit/PR；
- 环境有可重放Dockerfile/script/manifest/lock或derived image digest；
- actual execution code identity与Git指针一致；
- Code、Control、Evidence三指针完整。

## STOP

如果CANN 9.0.0出现明确且无法合理适配的版本blocker，或合理调查后仍无法闭合，STOP并报告：

- 实际做到哪一步；
- root cause与repro；
- 尝试过的依赖、build、toolchain和patch；
- 已PASS门禁；
- 剩余blocker；
- 3个历史问题的状态；
- 推荐下一路线和需要的Decision。

未经Decision不得切换到CANN9.0.1。

## 禁止

- 修改Host全局Python/CANN/Driver；
- 覆盖official base或影响现有任务；
- 模型加载、vLLM serve、HCCL/TP、KV cache、完整Worker/ModelRunner runtime、benchmark/profile；
- 删除/覆盖现有Evidence、legacy、artifacts或长期repo；
- direct push integration branch、force push；
- 把exploratory dirty或uncommitted FL patch当正式Evidence。

结束后生成immutable`results/A2-V024-CLEANROOM-CANN900-PY311/<run-id>.md`并更新`results/INDEX.md`。Control non-fast-forward允许fetch/rebase安全重试；真实冲突时Control Sync=PENDING，实验PASS/STOP不受影响。
