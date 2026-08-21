# GLM-5.2-W8A8 × FlagOS × A3/910C 基线重置交付索引

## 文件

- `ENVIRONMENT-RESEARCH.md`：官方A3 CI、`ascend+empty`、vllm-ascend环境carrier与FlagOS runtime ownership证据。
- `FLAGOS-RUNTIME-MAP.md`：真实调用链与runtime ownership matrix。
- `COMPATIBILITY-MATRIX.md`：组件版本、910C成熟度、GLM-5.2-W8A8兼容矩阵与容量。
- `PLAN.md`：新阶段、Ready/Exit、owner、证据和第二台服务器触发。
- `STATUS.md`：当前状态、阻塞、GitHub状态与下一门禁。
- `DECISIONS.md`：已提议/待确认/待ADR技术决策。
- `MINIMAL-EAGER-EXECUTION-CLOSURE.md`：首次正确GLM-5.2-W8A8 eager token的mandatory能力闭包。
- `REPOSITORY-PLAN.md`：建仓前legacy保全与formal-fork/standalone路线分析；当前已被standalone决策取代。
- `CODE-REPOSITORY-BASELINE.md`：正式standalone A3代码仓库、official冻结SHA/tree、remote与sync policy、legacy零变化验收。
- `R0-CONTAINER-TUPLE-RESOLUTION.md`：`c70aa4b`时期neutral-base tuple的兼容性研究；其强制路线已Superseded，仅保留reference evidence。
- `tasks/STAGE-A-CLEAN-PROVENANCE.md`：按official carrier FL-only bring-up重定义后的Stage A父合同。
- `tasks/STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`：下一条建议Stage；一次性container受控卸载、minimal negative check与synthetic NPU operator smoke。
- `tasks/STAGE-A2-FLAGOS-RUNTIME-PROVENANCE-TRACE.md`：原pre-canary A2的历史指针，已Superseded为后置审计。
- `tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`：保留完整dynamic provenance设计；位于Eager Correctness之后、Baseline Benchmark之前。
- `tasks/STAGE-A1-A3-READ-ONLY-ENVIRONMENT-INVENTORY.md`：历史/补证用只读inventory合同；当前tuple决策已不以其为前置，状态Not Ready。
- `tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md`：gap confirmation、最小能力实现、microgate PASS与vLLM-Ascend reference规则。

## 给用户的10条摘要

1. 官方910C路线确实以vllm-ascend A3 image作环境carrier，package/custom artifacts仍在，CI没有卸载它；存在本身不再判违规。
2. `VLLM_PLUGINS=fl`激活`PlatformFL`；静态代码继续进入`WorkerFL`、`ModelRunnerFL`与FlagOS Dispatch。
3. 当前先做FL-only最小smoke；完整关键operator、`vllm_ascend`动态import/call和native provenance审计后置到Eager Correctness之后。
4. `vendor.ascend`是`vllm_fl`自有backend，部分文件注明adapted来源但运行时直接调用torch_npu/CANN；它不是vllm-ascend backend wrapper。
5. 首个严格910C-backed canary是Qwen3.6-27B TP2 eager，README里的更小Qwen不能写成官方910C已验证。
6. GLM-5.2至少被vLLM0.20.2语义、sparse MLA/Indexer、W8A8 Linear和ModelSlim格式四类缺口阻塞。
7. ModelSlim能生成A3 W8A8不等于FL能加载；AscendV1 reader当前只在vllm-ascend中找到，其是否需要迁入仍由artifact contract决定。
8. 一台A3官方物理规格为8×128GB，当前Host边界确认16×64GB logical devices；full-model容量仍必须由container device trace与真实checkpoint manifest计算，现在不需要第二台。
9. 正式代码仓库已采用personal standalone方案并精确复制official冻结main；legacy、PR #1、branches、tags和settings保持零变化。
10. 下一步建议是Official Carrier FL-only Environment Smoke；container内卸载只是减少变量，不恢复“package存在即违规”的旧规则。

## 状态标签

- Confirmed：直接官方源码、模型卡、OCI、CI job或产品规格支持。
- Inferred：由Confirmed证据推导，但未在目标container/现场实验验证。
- Unknown：公开证据不足。
- Missing：当前目标基线明确无实现、显式报错或缺必需接口。
- Conflicting：可信来源在版本/路线/行为上不一致，尚未由实验消解。

本目录位于项目唯一控制仓库`yanceng305-collab/glm52-w8a8-a3-flagos-control`。正式A3代码仓库已按`CODE-REPOSITORY-BASELINE.md`落地；服务器、DeepSeek、A2 smoke和实现仍未启动。
