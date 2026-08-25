# GLM-5.2-W8A8 × FlagOS × A3/910C 基线重置交付索引

## 文件

- `ENVIRONMENT-RESEARCH.md`：官方A3 CI、`ascend+empty`、vllm-ascend环境carrier与FlagOS runtime ownership证据。
- `FLAGOS-RUNTIME-MAP.md`：真实调用链与runtime ownership matrix。
- `COMPATIBILITY-MATRIX.md`：组件版本、910C成熟度、GLM-5.2-W8A8兼容矩阵与容量。
- `PLAN.md`：新阶段、Ready/Exit、owner、证据和第二台服务器触发。
- `STATUS.md`：当前状态、阻塞、GitHub状态与下一门禁。
- `DECISIONS.md`：已提议/待确认/待ADR技术决策。
- `REPOSITORY-AND-EVIDENCE-RULES.md`：Control/Code/Server Evidence、角色、commit与一任务三指针规则。
- `results/INDEX.md`：DeepSeek immutable run snapshot索引与Codex acceptance状态。
- `MINIMAL-EAGER-EXECUTION-CLOSURE.md`：首次正确GLM-5.2-W8A8 eager token的mandatory能力闭包。
- `REPOSITORY-PLAN.md`：建仓前legacy保全与formal-fork/standalone路线分析；当前已被standalone决策取代。
- `CODE-REPOSITORY-BASELINE.md`：正式standalone A3代码仓库、official冻结SHA/tree、remote与sync policy、legacy零变化验收。
- `OFFICIAL-V024-BASELINE-RESEARCH.md`：official branch migration、new main/vLLM0.24、GLM partial、v0.24 carrier、双device与compiler证据。
- `CODE-REPOSITORY-MIGRATION-V024.md`：不修改existing main的0.24 baseline/project branch迁移记录；已PASS。
- `A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md`：accepted runtime tuple、现场Docker参数、default local snapshot与明确的非阻塞replay文档债。
- `R0-CONTAINER-TUPLE-RESOLUTION.md`：`c70aa4b`时期neutral-base tuple的兼容性研究；其强制路线已Superseded，仅保留reference evidence。
- `tasks/STAGE-A-CLEAN-PROVENANCE.md`：因branch migration重开的v0.24 baseline/carrier/provider Stage A父合同。
- `tasks/STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`：historical v0.20.2 A2；已Paused，不得执行。
- `tasks/DEEPSEEK-A3-CP-A2-EXECUTION-PROMPT.md`：historical v0.20.2 prompt；已Paused，不得下发。
- `tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE-DRAFT.md`：v0.24 A2历史draft；已被final contract取代。
- `tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`：A2合同；当前Paused。
- `tasks/DEEPSEEK-A3-CP-A2-V024-EXECUTION-PROMPT.md`：A2执行提示词；当前Paused，不得下发。
- `tasks/DEEPSEEK-A3-HOST-DIRECTORY-INITIALIZATION-PROMPT.md`：Host目录/repo初始化prompt；latest run已ACCEPTED with deviation。
- `tasks/STAGE-A2-V024-FLAGTREE-PY312-INTEGRATION-GAP.md`与对应prompt：已被Python3.11-first路线Superseded。
- `tasks/STAGE-A2-V024-PY311-FIRST-FLAGTREE-ENV.md`与对应prompt：feasibility run已完成；不作formal acceptance，不得重跑。
- `tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`与对应旧prompt：execution PASS reported，保留为原始合同，不得重跑。
- `tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION.md`：Completed；run `20260825T030520Z` PASS / Codex1 ACCEPTED（A2 scope-limited）。
- `tasks/CODEX2-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION-PROMPT.md`：已执行的Codex2入口；不得重新下发。
- `tasks/DEEPSEEK-A3-CP-A2-V024-PHASE-A-EXECUTION-PROMPT.md`：旧Phase-A-only提示词；已Superseded，不得执行。
- `tasks/STAGE-A2-FLAGOS-RUNTIME-PROVENANCE-TRACE.md`：原pre-canary A2的历史指针，已Superseded为后置审计。
- `tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`：保留完整dynamic provenance A/B设计；Eager Correctness后的Deferred / On-demand支线，不阻塞Baseline Benchmark。
- `tasks/STAGE-A1-A3-READ-ONLY-ENVIRONMENT-INVENTORY.md`：历史/补证用只读inventory合同；当前tuple决策已不以其为前置，状态Not Ready。
- `tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md`：gap confirmation、最小能力实现、microgate PASS与vLLM-Ascend reference规则。

## 给用户的10条摘要

1. Official branch migration现定义`v0.2.1 = vLLM0.20.2 maintenance`、`main = vLLM0.24 current`；observed main为`a9435a34...`。
2. 正式repo迁移已PASS：existing main不变，v0.2.1 anchor、0.24 anchor与project branch精确创建。
3. Primary GLM contract改为FlagOS new main + vLLM0.24 + GLM-5.2-W8A8；0.20.2只作fallback evidence。
4. `bb439d...`已在NVIDIA TP16双机完成GLM-5.2-Slim init/weight load并暴露MLA cache gap，但不是Ascend/W8A8 E2E。
5. 当前A2正式base为official CANN9.0.0 A3 py311 exact digest `sha256:5f20011b...`；旧vLLM-Ascend carrier只保留history/reference。
6. Accepted tuple为Python3.11.15 / CANN9.0.0 / torch2.10 / torch_npu2.10 / vLLM0.24 empty / FlagTree0.6.1+ascend3.5 / FlagGems5.3.4 / FL`a9435a34...`。
7. FlagTree single-provider与torch_npu、FlagTree、FlagGems direct、FL/Dispatch四项explicit NPU tiny path均已ACCEPTED；这不是模型级E2E。
8. Current main Ascend Dockerfile仍为0.19/CANN8.5 old tuple，明确标记Upstream Conflict / stale candidate。
9. Original run `20260824T080753Z`、supplement与final verification `20260825T030520Z`已联合ACCEPTED；Experiment Result、Control Sync与Codex Acceptance继续分开记录。
10. A2不再阻塞下一Stage选择与contract设计；当前无下一task被创建、Ready或dispatch，GLM模型级工作仍未开始。

## 状态标签

- Confirmed：直接官方源码、模型卡、OCI、CI job或产品规格支持。
- Inferred：由Confirmed证据推导，但未在目标container/现场实验验证。
- Unknown：公开证据不足。
- Missing：当前目标基线明确无实现、显式报错或缺必需接口。
- Conflicting：可信来源在版本/路线/行为上不一致，尚未由实验消解。

本目录位于项目唯一控制仓库`yanceng305-collab/glm52-w8a8-a3-flagos-control`。A2 clean-room environment/runtime/compiler/Dispatch tiny gate已ACCEPTED。下一Stage选择与contract设计已解锁，但当前没有下一task被创建、Ready或dispatch。
