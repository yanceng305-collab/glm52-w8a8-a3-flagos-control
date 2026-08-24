# DeepSeek Prompt — A2-V024-FLAGTREE-PY312-GAP

状态：**Ready after user dispatch**

唯一合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-FLAGTREE-PY312-INTEGRATION-GAP.md`

## 目标

调查并尽可能解决：

```text
vLLM 0.24 A3 carrier
  × Python 3.12.13
  × FlagTree ascend3.5
  × FlagGems / vllm-plugin-FL / FlagOS Dispatch
```

让后续A2能够越过FlagTree gate。不要预先假定必须源码构建、必须使用某个wheel，也不要未经Decision切换到Python3.11 carrier。

## 固定事实

- Carrier：`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`
- Measured Python：3.12.13
- Code repo：`/data/tiankuan/zyg/repos/vllm-plugin-FL-a3-flagos/`
- Control repo：`/data/tiankuan/zyg/repos/glm52-w8a8-a3-flagos-control/`
- Code base：branch=`project/glm52-w8a8-v024`，commit=`a9435a34dcd7d0a38e3a853535947371a6c62205`，tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- Previous result：`results/A2-v024/20260824T025250Z.md`
- Task ID：`A2-V024-FLAGTREE-PY312-GAP`
- Run ID：UTC timestamp
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-FLAGTREE-PY312-GAP/<run-id>/`

## 执行原则

- 先同步并验证两个长期repo；不使用`/root/vllm-plugin-FL/`或legacy副本作为源码。
- 复现previous FlagTree failure，然后从network availability、artifact availability、Python tag/ABI、pybind11、build-system、toolchain、native extension和provider overlay逐层定位。
- 你可以自主访问排障所需的公开/官方文档、源码、package index、registry、CI artifact和合理镜像；可在disposable container/work目录安装build/debug依赖、调整构建工具、构建wheel、检查其他FlagOS image/artifact并做临时patch。
- 不采用固定命令清单。记录关键命令、URL、ref/version、artifact SHA256、退出码和raw logs；不得泄露credential。
- 所有实验写入`work/A2-V024-FLAGTREE-PY312-GAP/<run-id>/`、`artifacts/`或本次Evidence；不得修改Host全局Python/CANN/Driver。
- Exploratory dirty working tree允许；正式PASS/STOP前，actual execution source必须绑定exact repo/branch/commit/tree。若修改vllm-plugin-FL，创建`task/A2-V024-FLAGTREE-PY312-GAP`，commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- 不向第三方repo push。FlagTree临时patch/source/wheel保存在work/artifacts并记录provenance。

## 最小闭环

1. 重放并保存原始failure。
2. 比较可行路线，选择证据最强、对当前carrier改动最小的一条实施。
3. 在exact carrier中安装身份明确的FlagTree ascend3.5，证明single coherent provider，无混合RECORD/module/native tree。
4. 验证FlagTree import、Ascend backend/driver origin和最小kernel compile。
5. 安装/验证exact FlagGems与FL，完成最小Dispatch integration。
6. 如需NPU验证，重新检查共享资源后仅使用NPU 12+13执行极小tensor、FlagTree kernel和一个FlagOS Dispatch operator；禁止collective。

## PASS

只有以下全部成立才报告PASS：

- Python3.12 carrier中有可重复安装、SHA256固定的FlagTree ascend3.5 artifact；
- provider distribution/RECORD/module/native/backend origin一致；
- 最小FlagTree kernel在Ascend成功compile/execute；
- FL/FlagGems/Dispatch smoke通过，tensor留在NPU且无silent CPU fallback；
- 安装/构建可重放，一任务三指针完整。

## STOP / PARTIAL

- 无法在Python3.12闭合时报告STOP：给出最小root cause、已排除路线、repro和Decision Request；不要把“当前artifact不可用”扩大成“FlagTree不支持Python3.12”。
- 只证明路线可行但未形成可重放artifact或未完成NPU闭环时报告PARTIAL。
- 若唯一可行方向要求Python3.11 carrier、改变primary runtime、Host全局修改或模型级验证，停止并请求Decision，不自行切换路线。

## 禁止

- GLM/Qwen模型加载、vLLM serve、HCCL/TP、KV cache、完整Worker/ModelRunner runtime、benchmark/profile；
- 删除/覆盖现有Evidence、legacy或长期repo；
- direct push integration branch、force push、把exploratory dirty运行当正式Evidence。

结束后生成immutable `results/A2-V024-FLAGTREE-PY312-GAP/<run-id>.md`并更新`results/INDEX.md`。Control non-fast-forward允许fetch/rebase安全重试；真实冲突时Control Sync=PENDING，实验PASS/STOP不受影响。
