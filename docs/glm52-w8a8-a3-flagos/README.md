# GLM-5.2-W8A8 × FlagOS × A3/910C 基线重置交付索引

## 文件

- `ENVIRONMENT-RESEARCH.md`：官方A3 CI、`ascend+empty`、vllm-ascend镜像角色与clean-room路线。
- `FLAGOS-RUNTIME-MAP.md`：真实调用链与runtime ownership matrix。
- `COMPATIBILITY-MATRIX.md`：组件版本、910C成熟度、GLM-5.2-W8A8兼容矩阵与容量。
- `PLAN.md`：新阶段、Ready/Exit、owner、证据和第二台服务器触发。
- `STATUS.md`：当前状态、阻塞、GitHub状态与下一门禁。
- `DECISIONS.md`：已提议/待确认/待ADR技术决策。
- `MINIMAL-EAGER-EXECUTION-CLOSURE.md`：首次正确GLM-5.2-W8A8 eager token的mandatory能力闭包。
- `REPOSITORY-PLAN.md`：建仓前legacy保全与formal-fork/standalone路线分析；当前已被standalone决策取代。
- `CODE-REPOSITORY-BASELINE.md`：正式standalone A3代码仓库、official冻结SHA/tree、remote与sync policy、legacy零变化验收。
- `tasks/STAGE-A-CLEAN-PROVENANCE.md`：首个可执行Stage的任务合同和验收。
- `tasks/STAGE-A1-A3-READ-ONLY-ENVIRONMENT-INVENTORY.md`：第一台A3服务器的严格只读inventory Ready task与DeepSeek完整提示词。
- `tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md`：gap confirmation、最小能力实现、microgate PASS与vLLM-Ascend reference规则。

## 给用户的10条摘要

1. 官方910C CI确实基于vllm-ascend A3镜像，package/custom kernel仍在，CI没有卸载它。
2. 但实际激活的是FL platform；vllm-ascend entry point只是可发现并被`VLLM_PLUGINS=fl`过滤。
3. 官方CI不能证明“完全没有vllm-ascend也能跑”；客户合规路线必须从neutral CANN base零起做负证据验收。
4. Actual CI用triton-ascend 3.2.1而非FlagTree；两者先分profile，FlagCX也后置，baseline先走HCCL。
5. 首个严格910C-backed canary是Qwen3.6-27B TP2 eager，README里的更小Qwen不能写成官方910C已验证。
6. GLM-5.2至少被vLLM0.20.2语义、sparse MLA/Indexer、W8A8 Linear和ModelSlim格式四类缺口阻塞。
7. ModelSlim能生成A3 W8A8不等于FL能加载；AscendV1 reader只在被禁的vllm-ascend中。
8. 一台A3官方物理规格为8×128GB；runtime logical topology和full-model容量必须由现场inventory与真实checkpoint manifest计算，现在不需要第二台。
9. 正式代码仓库已采用personal standalone方案并精确复制official冻结main；legacy、PR #1、branches、tags和settings保持零变化。
10. 下一步不是写GLM补丁，而是确认边界后完成`R0-clean`、Qwen canary，再做vLLM与量化contract决策。

## 状态标签

- Confirmed：直接官方源码、模型卡、OCI、CI job或产品规格支持。
- Inferred：由Confirmed证据推导，但未在目标clean-room/现场实验验证。
- Unknown：公开证据不足。
- Missing：当前目标基线明确无实现、显式报错或缺必需接口。
- Conflicting：可信来源在版本/路线/行为上不一致，尚未由实验消解。

本目录位于项目唯一控制仓库`yanceng305-collab/glm52-w8a8-a3-flagos-control`。正式A3代码仓库已按`CODE-REPOSITORY-BASELINE.md`落地；服务器、DeepSeek、Clean Provenance和实现仍未启动。
