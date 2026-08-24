# GLM-5.2-W8A8 × FlagOS × Ascend A3/910C 项目计划

状态：A3 Host directory initialization ACCEPTED with deviation；FlagTree Python3.12 integration-gap task Ready
基线调查日期：2026-08-21
正式代码repo existing `main@92a6f767...`保持v0.2.1 maintenance/reference；immutable 0.24 baseline与`project/glm52-w8a8-v024`已创建于`a9435a34...`/tree`e5e073ed...`。

## 结果目标

在 Ascend A3/910C 上遵循official FlagOS Ascend实际环境与插件路线，以可审计的运行时证据证明模型执行由`PlatformFL -> WorkerFL -> ModelRunnerFL -> FlagOS Dispatch`拥有，关键算子落到FlagGems、`vendor.ascend`或NPU-resident Reference，再进入torch_npu/CANN/910C；在此基础上完成GLM-5.2-W8A8正确运行与逐阶段性能优化。

## 硬约束

- 新代码基线只从官方 current `main` 开始；legacy A2 仓库、branch、PR #1、Stage 0 结论只保留历史，不作新基线。
- `92a6f767...`不得删除、force覆盖或与new main强行合并；official `v0.2.1`与new `main`已diverged。
- vllm-ascend image/package的**存在性不再自动判违规**。official同款A3 image可以作为环境carrier；是否存在不可接受依赖必须依据runtime import/call、entry-point activation、operator/backend ownership和loaded-library trace判定。
- 正式模型执行必须由FlagOS runtime/dispatch/backend ownership闭合。若trace发现`vllm_ascend`实际参与执行，先记录调用点、作用和必要性，再由control判断客户边界与是否替换；不得用“package已安装”或“未发现静态import”替代运行时证据。
- A3-CP-A2允许只在新建的一次性实验container内卸载`vllm-ascend`，用于降低FL-only bring-up变量；这不是package-presence合规门禁，也不否定official coexistence路线。不得修改原始image或其他carrier runtime组件。
- `118c314`中的old v0.20 A2/prompt继续Paused；new v0.24 A2 latest run已STOP。下一步只调查/解决FlagTree Python3.12 integration gap，不恢复完整A2。
- Repository、角色、immutable result、Control Sync和正式Evidence identity规则以`REPOSITORY-AND-EVIDENCE-RULES.md`为准。
- README、代码、Docker/CI、模型卡冲突必须保留；Unknown 不补猜测版本。
- eager correctness 在先；graph、MTP、multistream、FlagCX、多机和组合优化在后。
- 目标模型固定为GLM-5.2-W8A8；W8A8是首次目标模型eager correctness硬门禁，不是性能优化。BF16只允许operator/reference/debug microtest，不能替代目标bring-up。
- FlagScale暂不作为首次模型bring-up前置；先直接闭合`vLLM -> vllm-plugin-FL -> PlatformFL -> WorkerFL -> ModelRunnerFL -> FlagOS Dispatch`，模型推理链稳定后才在现有后期集成阶段验证FlagScale。
- FlagGems是preferred实现来源而非mandatory依赖。Bring-up优先correctness和FlagOS Dispatch可达性，不以统一算子来源或FlagGems覆盖率作为首次eager门禁。
- A2 environment preflight不映射NPU；通过后允许共享NPU 12+13做极小`torch_npu` tensor与FlagOS Dispatch operator smoke，不要求完全空闲pair。共享资源状态恶化立即STOP。
- FlagTree rc1可能与carrier内实际compiler/provider共享`triton`namespace；A2必须先做实际inventory，再在disposable container内执行package-manager replacement并验证single coherent provider，失败即STOP。不得预设carrier初始provider或版本。
- New-main FL安装必须从readonly source的writable `.git`副本执行`--no-build-isolation --no-deps -e`；缺build requirement时STOP，不得联网补包。
- A2 carrier只接受D-062 exact digest；网络只允许FlagTree official resource index和official FlagGems v5.3.4，禁止其他image/tag/index和核心runtime升级。
- FlagGems只允许exact source + current container Python + `--no-build-isolation --no-deps`安装；禁止`setup.sh`、`flaggems-setup`和任何bootstrap/独立环境流程。
- 服务器缺少formal FL source时可direct clone唯一repo/project branch；GitHub不可达时允许expected SHA256校验过的Git bundle relay到Evidence source目录，最终验证exact branch/HEAD/tree/clean。
- GitHub是唯一项目事实源：本控制仓库管理PLAN/DECISIONS/tasks，正式代码仓库及冻结关系以`CODE-REPOSITORY-BASELINE.md`为准。

## 当前关键路径

```text
A3 Host directory / long-term Git working tree initialization (ACCEPTED)
  -> FlagTree Python3.12 integration gap (Ready)
  -> reproducible artifact + minimal integration smoke
  -> Codex acceptance / route decision
  -> A3-CP-A2-v024 environment identity/preparation
  -> shared NPU tiny smoke
  -> 910C Canary (v0.24 control-approved model TBD)
  -> GLM Contract Gate (vLLM语义 + quant format + Minimal Eager Execution Closure)
  -> Capability Microgates / Gap Confirmation
  -> Minimal Capability Implementation
  -> Corresponding Microgate PASS
  -> Capacity & Placement
  -> First Eager Load
  -> Minimal Compatibility
  -> Eager Correctness
  -> Baseline Benchmark
  -> Profile & Bottleneck
  -> Single-variable Optimize
  -> Advanced Composition
  -> Scale-out Acceptance（仅被触发时）

Deferred / on-demand side branch after Eager Correctness:
  -> Post-Eager Runtime Provenance Audit
```

## 阶段计划

| Stage | 目标 | Ready gate | Exit / 验收 | 必存证据 | Owner | 第二台 A3 |
|---|---|---|---|---|---|---|
| **v0.24 Baseline Refresh** | 冻结branch语义、new main SHA/tree、carrier tuple、compiler冲突和GLM evidence | developer通知 + official源码 | **PASS**；official refs frozen | SHA/tree、tuple、conflicts、Unknown | Codex | 不需要 |
| **Formal Code Repository Migration** | 不改existing main，新增0.24 immutable baseline与project branch | 用户批准 | **PASS**；三个refs exact，existing main/default/legacy零变化 | refs/SHA/tree/legacy hashes | Codex | 不需要 |
| **A3 Host directory initialization** | 建立两个长期Git working tree和repos/evidence/artifacts/work/legacy边界 | **ACCEPTED with deviation** | repo identity/clean PASS；legacy复制偏差记录，原目录未改 | immutable result + acceptance index | DeepSeek执行；Codex验收 | 未访问NPU |
| **FlagTree Python3.12 integration gap** | 在exact vLLM0.24 carrier中闭合FlagTree ascend3.5 artifact/provider/minimal execution | **Ready after user dispatch** | reproducible artifact、single provider、minimal kernel/Dispatch PASS，或exact STOP/Decision Request | source/artifact provenance、build/install logs、Code/Control/Evidence三指针 | DeepSeek执行；Codex验收 | 必要时仅tiny NPU |
| **A3-CP-A2-v024 environment gate** | 无NPU映射完成carrier/source/package/compiler/FlagGems/FL准备与静态验证 | **PAUSED**；等待integration-gap结果 | gap accepted且用户授权后重试 | immutable result + Server Evidence | DeepSeek未来执行；Codex验收 | 不需要NPU |
| **A3-CP-A2-v024 shared-NPU tiny smoke** | 在受限新container完成最小NPU/Dispatch验证 | **Not Ready**；environment gate未通过且A2暂停 | tiny torch_npu与一个Dispatch op PASS | environment引用 + NPU pre/post + Dispatch/device result | DeepSeek未来执行；Codex验收 | 第一台NPU 12+13 |
| **910C Canary** | 用new v0.24 stack上的control-approved小模型隔离验证基础链 | A3-CP-A2-v024 accepted；canary模型/权重重新冻结 | eager offline + serving正确；FL/dense attention/HCCL dispatch可追溯 | prompts/outputs、tolerance、dispatch trace、峰值内存 | DeepSeek；Codex验收 | 不需要 |
| **GLM Contract Gate** | 冻结`FlagOS new main + vLLM0.24 + GLM-5.2-W8A8`模型与artifact contract | Canary accepted；真实checkpoint manifest齐全 | 验证current-main GLM contract、W8A8 artifact、MLA/DSA/Indexer/W8A8 Linear/MoE、MLA cache ops与910C A/B/C closure；0.20.2仅fallback evidence | current-main code map、manifest/layout、gap contracts、closure证据 | Codex决策；DeepSeek仅做未来授权spike | 不需要 |
| **Capability Microgates / Gap Confirmation** | 对每个mandatory capability按`FlagGems -> vendor.ascend -> Reference/PyTorch`依次审查，确认现有合法路径或形成gap contract | 两项contract ADR和Minimal Eager Execution Closure批准 | 路径在FlagOS Dispatch内可达、910C可执行、microgate correctness通过、接口支撑GLM forward；Reference须证明tensor留在NPU且无静默CPU fallback；任何`vllm_ascend`实际调用须可追踪并进入边界审查；三路都失败才标Missing/Unwired | path audit、gap contract、reference/tolerance、device/backend/import trace、failure signature | Codex定义/审查；DeepSeek仅在未来授权后执行repro | 不需要 |
| **Minimal Capability Implementation** | 只补齐A/B/C三条现有路径全部不可用且属于首次eager mandatory closure的能力 | 对应gap contract证明三路均失败并获批准 | MLA、DSA/SFA、Indexer、W8A8等原则上独立小任务、独立branch、独立Draft PR；按FlagOS架构spec-first重新实现；不为FlagGems覆盖率或性能提前开发 | implementation contract、path audit、PR diff、license/reference disclosure、focused tests | DeepSeek未来实现；Codex contract/PR review | 不需要 |
| **Corresponding Microgate PASS** | 对选定现有路径或最小新实现使用同一contract验收 | 对应合法路径可测试或独立Draft PR可测试 | backend可达、小shape correctness、checkpoint/runtime contract、GLM forward接口全部PASS；device/backend trace证明NPU执行且无静默CPU fallback；全部mandatory项PASS后才解锁Capacity | raw test、reference/tolerance、dispatch/device trace、contract audit | DeepSeek未来测试；Codex逐项验收 | 不需要 |
| **Capacity & Placement** | 基于真实artifact与已冻结16×64GB Host topology确定布局 | checkpoint manifest、container内`npu-smi`/`torch.npu.device_count()`/device properties、目标上下文/并发、**全部mandatory microgate PASS** | 验证16-device可见性并按真实packed weights/scales/float tensors/workspace/KV/communication headroom完成per-rank预算；冻结首次load的TP/EP候选与安全余量 | manifest audit、placement simulation、memory budget、container device trace | DeepSeek测算；Codex验收 | 默认不需要；不足则触发 |
| **First Eager Load** | 使用真实GLM-5.2-W8A8 checkpoint完成首次capacity-valid load，收敛首个真实故障 | placement accepted；**全部mandatory microgate PASS** | 模型构造/权重load成功，或只保留一个可复现first failure；实际走W8A8 Linear/MoE与MLA+DSA+Indexer；关闭MTP/graph/FlagCX/multistream | exact run config、first-error log、weight/scale-key audit、backend trace | DeepSeek；Codex定下一任务 | 仅Capacity明确触发；开始前必须通知 |
| **Minimal Compatibility** | 只处理First Eager Load后新暴露的整模型问题，不接收已知mandatory capability补齐 | First Eager Load产生此前microgate无法暴露的单一failure | 一次一个新故障、最小patch、focused test、mandatory microgate与Qwen canary无回归、独立Draft PR | first-failure证据、diff、tests、provenance、rollback | DeepSeek；Codex PR review | 沿用批准布局 |
| **Eager Correctness** | 用真实GLM-5.2-W8A8 checkpoint完成capacity-valid eager正确性 | load/forward blockers关闭 | 第一个正确token及固定数据集满足reference/tolerance；W8A8 Linear/MoE、MLA、DSA/Indexer owner/shared和quant scales均通过；BF16不得替代 | golden/tolerance、layer/backend trace、weight/scale audit、memory/KV | DeepSeek；Codex阶段验收 | 仅容量需要 |
| **Baseline Benchmark** | 建立未优化基线 | Eager Correctness accepted；Post-Eager Audit不是默认前置 | 固定输入分布/上下文/并发；给 TTFT、TPOT、吞吐、延迟分位、利用率、功耗、内存和方差 | raw results、env SHA、重复运行 | DeepSeek | 单机先 |
| **Post-Eager Runtime Provenance Audit（Deferred / On-demand支线）** | 对比official carrier coexistence与同carrier FL-only路线，完整审计动态ownership | Eager Correctness accepted；客户要求证明coexistence、正式方案考虑保留distribution、A/B出现行为差异、或最终交付需要完整provenance之一触发；A/B合同另行批准 | Platform/Worker/Runner/Dispatch、GLM关键operator、`vllm_ascend`动态参与、compiler/provider与native library路径均有范围限定结论 | A/B manifest、import/call/library trace、per-op backend、provider/CANN trace、审计矩阵 | DeepSeek未来执行；Codex审计/裁决 | 默认不需要 |
| **Profile & Bottleneck** | 证明首要瓶颈 | baseline稳定 | kernel/communication/host/调度归因；每项有可证伪假设 | profiler trace、timeline、算子/通信占比 | DeepSeek；Codex审查 | 仅证据指向 scale-out 时触发 |
| **Single-variable Optimize** | 每次只改变一个算子或配置 | profile假设批准 | 正确性无回归，收益跨多轮成立；失败实验也留证 | before/after、方差、PR、rollback | DeepSeek；Codex验收 | 按实验合同 |
| **Advanced Composition** | graph、MTP、multistream、FlagTree、FlagCX、FlagScale、TP/EP/DP独立后再组合 | eager与单变量结果稳定 | ablation能归因；组合无correctness/stability回归；FlagScale不反向成为模型bring-up依赖 | ablation matrix、fallback、稳定性 | DeepSeek；Codex阶段验收 | 仅 multi-node 项需要 |
| **Scale-out Acceptance** | 两机 capacity/communication/生产负载验收（若需要） | 明确触发原因、网络与客户通信约束冻结 | HCCL baseline 和可选 FlagCX 分开验证；SLO、稳定性、故障恢复满足验收 | topology、collective、E2E benchmark、故障记录 | DeepSeek；Codex/用户验收 | **此阶段开始需要第二台 A3 服务器** |

### Branch migration、A2与Runtime Provenance当前决策

`c70aa4b`的neutral-base-only门禁继续保持Superseded，不因本轮版本迁移恢复。`118c314`中的v0.20.2 A2及其prompt现为**Superseded / Paused by upstream branch migration**，不得执行。

Official documented release tag `quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`当前无可用artifact。A2 provisional carrier与已测runtime事实保留在immutable result中；完整A2 task/prompt仍Paused。当前Ready task是[`tasks/STAGE-A2-V024-FLAGTREE-PY312-INTEGRATION-GAP.md`](tasks/STAGE-A2-V024-FLAGTREE-PY312-INTEGRATION-GAP.md)，prompt见[`tasks/DEEPSEEK-A2-V024-FLAGTREE-PY312-INTEGRATION-GAP-PROMPT.md`](tasks/DEEPSEEK-A2-V024-FLAGTREE-PY312-INTEGRATION-GAP-PROMPT.md)。

完整coexistence/dynamic provenance不再阻塞Qwen、GLM mandatory closure、first eager或Baseline Benchmark；原trace设计移至[`tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`](tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md)，作为Eager Correctness后的Deferred / On-demand支线，按客户证明要求、正式方案保留distribution、A/B行为差异或最终交付provenance需求触发。A/B仍比较“保留package + FL selectors”和“同carrier卸载package”。Host/Container边界仍有效；Host CANN不参与tuple选择，除非显式bind-mount Host Toolkit。

## GLM Contract Gate — v0.24 primary

旧问题“0.20.2 backport还是0.23/0.24 uplift”已被branch migration Superseded。Primary contract固定为：

```text
FlagOS official new main
  + vLLM 0.24.0
  + GLM-5.2-W8A8 actual checkpoint
  + Ascend A3/910C
```

Gate必须重新核对：current-main model/config/IndexShare/MTP contract；真实W8A8 manifest与AscendV1/compressed-tensors格式；MLA；DSA/SFA；Indexer；W8A8 Linear/MoE；`concat_and_cache_mla`及fused RoPE variant；FlagGems/vendor.ascend/NPU-resident Reference三路可达性。`bb439d...`的NVIDIA PARTIAL只证明0.24构造/weight-load起点，不证明Ascend或W8A8 closure。

## Minimal Eager Execution Closure

官方config、Transformers v5.12.0和vLLM 0.23.0均把GLM-5.2的`index_topk/indexer_types`用于DSA与Indexer执行；没有官方支持且correctness等价的Dense MLA fallback。`VLLM_MLA_DISABLE`也不能形成合法GLM-5.2 fallback。故首次目标模型closure为：真实W8A8 checkpoint + W8A8 Linear/MoE + MLA + DSA/SFA + Indexer +必要MoE/runtime + eager。详细证据见[`MINIMAL-EAGER-EXECUTION-CLOSURE.md`](MINIMAL-EAGER-EXECUTION-CLOSURE.md)。

Microgates只证明backend可达、小规模correctness、checkpoint/runtime contract和目标forward接口；不要求graph、MTP、multistream、FlagCX、fusion或production性能。

Mandatory capability的统一Stage任务合同见[`tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md`](tasks/GLM-MANDATORY-CAPABILITY-CLOSURE.md)。该合同把gap confirmation、spec-first最小实现、对应microgate PASS和vLLM-Ascend技术参考边界合并为一条治理规则。

## Bring-up实现路径选择

每个mandatory capability/operator按以下顺序寻找现有实现：

1. 已有成熟FlagGems；
2. 已有`vendor.ascend`；
3. Reference/PyTorch通用实现；
4. 三者均不能满足microgate时，才进入Minimal Capability Implementation。

前三条任一只有同时满足以下条件才是合法bring-up路径：位于FlagOS Dispatch体系内；在910C执行；通过当前correctness microgate；接口支撑GLM-5.2 forward。Reference不是CPU fallback的同义词：tensor可留在NPU并由torch_npu/CANN执行，但必须保存device/backend trace，静默CPU fallback直接判失败。image/package的安装状态只作环境事实；若发现`vllm_ascend`实际import/call，则必须精确归因并由control裁定，不得默认为合规或违规。

Eager Correctness前不为了统一算子来源、提高FlagGems覆盖率或追求性能而开发新kernel。Eager Correctness后若profiling证明Reference为瓶颈，再分别比较FlagGems、vendor.ascend、Triton/FlagTree、fusion等单变量优化。

## vLLM-Ascend技术参考与实现边界

- 允许Codex深入研究官方vLLM-Ascend，提取Ascend/910C contract、shape/dtype/layout、KV Cache、prefill/decode、NPU primitive和correctness行为。
- 参考优先级固定为：FlagOS已有跨平台实现 → official vLLM/Transformers → vLLM-Ascend硬件实现参考。
- 生产实现必须进入`vLLM -> vllm-plugin-FL -> FlagOS dispatch/backend -> FlagGems/vendor.ascend/Reference -> torch_npu/CANN` ownership链；环境carrier可包含vllm-ascend distribution，但新能力不得绕过FlagOS ownership直接绑定vllm-ascend backend。发现实际`vllm_ascend`调用时先审查其职责和客户边界。
- 禁止机械复制vLLM-Ascend源码后改名、换目录或重构以隐藏来源。若确有源码复用/派生，必须遵守许可证并保留attribution，不能代码混淆规避声明。
- 每个capability Draft PR必须说明当前缺口、模型contract、hardware primitive、FlagOS ownership/dispatch、correctness test和vLLM-Ascend仅作为reference的具体范围。
- 当前客户核心边界解释为模型执行必须走FlagOS runtime/dispatch/backend ownership；不把carrier image/package存在性等同于backend依赖，也不扩大为禁止开发阶段研究源码。只有trace发现实际`vllm_ascend`参与，或客户以后明确禁止official FlagOS仓库中的历史adapted来源时，才对对应范围重新做合规判断。

## 阶段治理

- 每个实现/实验 Stage 使用独立 branch 和 Draft PR；不得自动 merge 或跨 Stage。
- Mandatory closure中的MLA、DSA/SFA、Indexer、W8A8等能力原则上分别使用独立任务、独立branch和独立Draft PR；不得把多个已知缺口打包成不可归因的大补丁。
- DeepSeek 只能执行控制面中的 `ready` task，不能修改 PLAN/STATUS/DECISIONS，不能自行进入下一 Stage。
- Codex 负责证据审查、技术决策、PR review、阶段验收与控制面更新。
- 任一新事实推翻路线时，保留已验证成果并回退到相应 Contract/Capability/Capacity gate，不全量重启。

## 两机触发规则

以下任一成立才允许申请第二台服务器：

1. 真实 checkpoint 的 per-rank placement + KV/workspace 安全余量证明单机不足；
2. 已进入明确的 HCCL/FlagCX、EP/DP 或 scale-out 对比实验；
3. 客户验收负载不能由单机覆盖。

任务 Ready 前必须直接写明：`此阶段开始需要第二台 A3 服务器`，并给用途、拓扑、通信 backend、停止条件和验收证据。
