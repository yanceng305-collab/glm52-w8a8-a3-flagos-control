# A2-V024-FLAGTREE-PY312-GAP — Python 3.12 × FlagTree ascend3.5 Integration

状态：**Ready after user dispatch**
执行对象：第一台Ascend A3/910C服务器

## 目标

调查并尽可能解决exact vLLM0.24 A3 carrier中Python 3.12.13与FlagTree ascend3.5的安装、ABI、provider与最小执行闭环，使后续A2能够越过FlagTree gate。

该任务不预选“构建Python3.12 FlagTree”或“切换Python3.11 carrier”路线。DeepSeek根据现场证据自主排障；需要改变primary carrier/Python合同的方案只形成Decision Request，不自行切换项目路线。

## 已确认起点

- Carrier：`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`
- Measured Python：3.12.13
- Code base：`project/glm52-w8a8-v024@a9435a34dcd7d0a38e3a853535947371a6c62205`，tree`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagTree target family：ascend3.5；prior intended version`0.6.1rc1+ascend3.5`
- Previous run：[`results/A2-v024/20260824T025250Z.md`](../results/A2-v024/20260824T025250Z.md)
- Long-term repos：`/data/tiankuan/zyg/repos/`

## 执行自主权

- 可访问完成排障所需的公开/官方文档、源码仓库、package index、container registry、CI artifact和合理镜像；不采用固定网络白名单。
- 可在disposable container与`work/<task-id>/<run-id>/`中安装合理build/debug依赖、调整构建工具、构建wheel、比较其他FlagOS artifact/image及进行临时source patch。
- 所有外部source/artifact必须记录URL、version/ref、SHA256和实际用途；不得暴露或写入credential。
- 不修改Host全局Python/CANN/Driver，不覆盖长期repo，不删除现有legacy/Evidence/artifact。

## Code与Evidence

- Exploratory阶段允许dirty worktree；正式PASS/STOP前必须绑定actual execution source的exact repo/branch/commit/tree。
- 若需要修改vllm-plugin-FL代码，从primary创建`task/A2-V024-FLAGTREE-PY312-GAP`，commit、push并创建base=`project/glm52-w8a8-v024`的PR。
- FlagTree临时patch/build保存在`work/`；可复用wheel/source archive进入`artifacts/`并记录SHA256。未经单独授权不向第三方repo push。
- Evidence：`/data/tiankuan/zyg/evidence/A2-V024-FLAGTREE-PY312-GAP/<run-id>/`。

## 调查与验证范围

1. 从previous run重放最小failure，区分网络不可达、wheel availability、Python tag/ABI、pybind11、build-system、toolchain和provider overlay问题。
2. 调查可行的Python3.12/ascend3.5 artifact或可复现构建路线；也可评估Python3.11路线，但不得未经Decision切换primary。
3. 在exact carrier的disposable container中形成一个single coherent FlagTree-owned provider；不得手工删除文件制造PASS。
4. 安装/验证exact FL与FlagGems identity，记录distribution、RECORD、module/native extension、backend/driver origin。
5. 如需硬件验证，可在重新检查共享资源安全后使用NPU 12+13执行极小tensor、FlagTree kernel和FlagOS Dispatch operator smoke；不执行collective。

## PASS

同时满足：

- exact Python3.12 carrier中可重复安装一个身份明确、SHA256固定的FlagTree ascend3.5 artifact；
- distribution/RECORD/module/native extension/backend origin一致，无混合provider；
- FlagTree import、Ascend backend发现与最小kernel compile/execute通过；
- FL/FlagGems/Dispatch最小集成smoke通过，输入输出保持NPU且无silent CPU fallback；
- 构建或安装步骤可重放，Code/Control/Evidence三指针完整。

## STOP / PARTIAL

- 如果合理调查后仍不能在Python3.12闭合，报告STOP并给出最小root cause、已排除路线、可复现失败和明确Decision Request。
- 如果只证明某条路线可行但尚未形成可重放artifact或NPU验证，报告PARTIAL。
- Python3.11 carrier、改变primary carrier/runtime、Host全局修改或扩大到模型推理均需上层Decision，不得自行执行。

## 禁止

- GLM/Qwen模型加载、vLLM serve、HCCL/TP、KV cache、完整Worker/ModelRunner runtime、benchmark/profile；
- 修改或删除现有Server Evidence、legacy目录、长期repo历史；
- direct push primary integration branch、force push或把exploratory dirty结果当正式Evidence。

任务结束后生成immutable result snapshot并更新`results/INDEX.md`；Control non-fast-forward按现行规则安全rebase或标记Control Sync PENDING，不改变实验结果。
