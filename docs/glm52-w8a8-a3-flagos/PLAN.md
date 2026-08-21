# GLM-5.2-W8A8 × FlagOS × Ascend A3/910C 项目计划

状态：Proposed for approval
基线调查日期：2026-08-21
代码候选起点：`flagos-ai/vllm-plugin-FL@38e7dbc20197e2db742c4e4c9687d36ea4df9900`（调查时 current `main` / `v0.2.1-rc0`）

## 结果目标

在 Ascend A3/910C 上建立一条从未安装、发现、导入、激活或链接 `vllm-ascend` image/package/runtime/plugin 的 FlagOS 正式环境，完成 GLM-5.2-W8A8 的正确运行与逐阶段性能优化，并以可复现环境、测试、benchmark、profile 和 Draft PR 证据完成验收。

## 硬约束

- 新代码基线只从官方 current `main` 开始；legacy A2 仓库、branch、PR #1、Stage 0 结论只保留历史，不作新基线。
- 正式环境禁止 vllm-ascend image/package/runtime/plugin；也禁止“先装/使用再卸载替换”。
- 当前阶段不操作服务器、不下发 DeepSeek、不编写 GLM 补丁或优化代码。
- README、代码、Docker/CI、模型卡冲突必须保留；Unknown 不补猜测版本。
- eager correctness 在先；graph、MTP、multistream、FlagCX、多机和组合优化在后。
- 目标模型固定为GLM-5.2-W8A8；W8A8是首次目标模型eager correctness硬门禁，不是性能优化。BF16只允许operator/reference/debug microtest，不能替代目标bring-up。
- FlagScale暂不作为首次模型bring-up前置；先直接闭合`vLLM -> vllm-plugin-FL -> PlatformFL -> WorkerFL -> ModelRunnerFL -> FlagOS Dispatch`，模型推理链稳定后才在现有后期集成阶段验证FlagScale。
- FlagGems是preferred实现来源而非mandatory依赖。Bring-up优先correctness和FlagOS Dispatch可达性，不以统一算子来源或FlagGems覆盖率作为首次eager门禁。
- GitHub 是唯一项目事实源；新仓库确认前，本文件只是 external-state candidate。

## 当前关键路径

```text
Research Freeze
  -> Clean Provenance (R0-clean，无 vllm-ascend)
  -> 910C Canary (Qwen3.6-27B TP2 eager)
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
```

## 阶段计划

| Stage | 目标 | Ready gate | Exit / 验收 | 必存证据 | Owner | 第二台 A3 |
|---|---|---|---|---|---|---|
| **Research Freeze** | 冻结官方 main、CI oracle、冲突和 Unknown | 本轮研究完成 | 用户批准研究结论、仓库方案和“原生”定义 | source SHA、CI links、矩阵、决策 | Codex；用户批准 | 不需要 |
| **Clean Provenance** | 从中性 CANN base 零起建立 `R0-clean` | 用户授权服务器；base policy、driver/CANN compatibility、repo 已确认 | vllm-ascend package/module/entry point/native lib 全部不存在；FL platform、torch-npu、HCCL、FlagOS Dispatch与backend inventory闭合；至少一条代表性合法NPU算子路径通过；fresh rebuild可复现 | inventory、negative audit、import/device/backend trace、原始日志、环境hash | DeepSeek 执行；Codex 验收 | 不需要 |
| **910C Canary** | 用官方当前 910C CI-backed 模型隔离验证基础链 | Clean Provenance accepted；Qwen3.6-27B 权重就绪 | TP2 eager offline + serving 正确；FL/dense attention/HCCL dispatch 可追溯；无污染依赖 | prompts/outputs、tolerance、dispatch trace、峰值内存 | DeepSeek；Codex验收 | 不需要 |
| **GLM Contract Gate** | 决定vLLM语义基线与W8A8 artifact contract，并冻结Minimal Eager Execution Closure | Canary accepted；真实checkpoint manifest齐全 | ADR选择`0.23/0.24 uplift`或`0.20.2 backport`；ADR选择`AscendV1 native loader`或经证明等价的`compressed-tensors conversion`；确认首次closure必须含W8A8+MLA+DSA/SFA+Indexer | API/worker diff、IndexShare ownership、tensor/scale/layout、closure证据 | Codex决策；DeepSeek仅做授权spike | 不需要 |
| **Capability Microgates / Gap Confirmation** | 对每个mandatory capability按`FlagGems -> vendor.ascend -> Reference/PyTorch`依次审查，确认现有合法路径或形成gap contract | 两项contract ADR和Minimal Eager Execution Closure批准 | 路径在FlagOS Dispatch内可达、无vllm-ascend依赖、910C可执行、microgate correctness通过、接口支撑GLM forward；Reference须证明tensor留在NPU且无静默CPU fallback；三路都失败才标Missing/Unwired | path audit、gap contract、reference/tolerance、device/backend trace、failure signature | Codex定义/审查；DeepSeek仅在未来授权后执行repro | 不需要 |
| **Minimal Capability Implementation** | 只补齐A/B/C三条现有路径全部不可用且属于首次eager mandatory closure的能力 | 对应gap contract证明三路均失败并获批准 | MLA、DSA/SFA、Indexer、W8A8等原则上独立小任务、独立branch、独立Draft PR；按FlagOS架构spec-first重新实现；不为FlagGems覆盖率或性能提前开发 | implementation contract、path audit、PR diff、license/reference disclosure、focused tests | DeepSeek未来实现；Codex contract/PR review | 不需要 |
| **Corresponding Microgate PASS** | 对选定现有路径或最小新实现使用同一contract验收 | 对应合法路径可测试或独立Draft PR可测试 | backend可达、小shape correctness、checkpoint/runtime contract、GLM forward接口全部PASS；device/backend trace证明NPU执行且无静默CPU fallback；全部mandatory项PASS后才解锁Capacity | raw test、reference/tolerance、dispatch/device trace、contract audit | DeepSeek未来测试；Codex逐项验收 | 不需要 |
| **Capacity & Placement** | 基于真实artifact与Stage A实测logical-device topology确定布局 | checkpoint manifest、`npu-smi`/`torch.npu.device_count()`/device properties、目标上下文/并发、**全部mandatory microgate PASS** | 按真实packed weights/scales/float tensors/workspace/KV/communication headroom完成per-rank预算；冻结首次load的TP/EP候选与安全余量 | manifest audit、placement simulation、memory budget、实测topology | DeepSeek测算；Codex验收 | 默认不需要；不足则触发 |
| **First Eager Load** | 使用真实GLM-5.2-W8A8 checkpoint完成首次capacity-valid load，收敛首个真实故障 | placement accepted；**全部mandatory microgate PASS** | 模型构造/权重load成功，或只保留一个可复现first failure；实际走W8A8 Linear/MoE与MLA+DSA+Indexer；关闭MTP/graph/FlagCX/multistream | exact run config、first-error log、weight/scale-key audit、backend trace | DeepSeek；Codex定下一任务 | 仅Capacity明确触发；开始前必须通知 |
| **Minimal Compatibility** | 只处理First Eager Load后新暴露的整模型问题，不接收已知mandatory capability补齐 | First Eager Load产生此前microgate无法暴露的单一failure | 一次一个新故障、最小patch、focused test、mandatory microgate与Qwen canary无回归、独立Draft PR | first-failure证据、diff、tests、provenance、rollback | DeepSeek；Codex PR review | 沿用批准布局 |
| **Eager Correctness** | 用真实GLM-5.2-W8A8 checkpoint完成capacity-valid eager正确性 | load/forward blockers关闭 | 第一个正确token及固定数据集满足reference/tolerance；W8A8 Linear/MoE、MLA、DSA/Indexer owner/shared和quant scales均通过；BF16不得替代 | golden/tolerance、layer/backend trace、weight/scale audit、memory/KV | DeepSeek；Codex阶段验收 | 仅容量需要 |
| **Baseline Benchmark** | 建立未优化基线 | Eager Correctness accepted | 固定输入分布/上下文/并发；给 TTFT、TPOT、吞吐、延迟分位、利用率、功耗、内存和方差 | raw results、env SHA、重复运行 | DeepSeek | 单机先 |
| **Profile & Bottleneck** | 证明首要瓶颈 | baseline稳定 | kernel/communication/host/调度归因；每项有可证伪假设 | profiler trace、timeline、算子/通信占比 | DeepSeek；Codex审查 | 仅证据指向 scale-out 时触发 |
| **Single-variable Optimize** | 每次只改变一个算子或配置 | profile假设批准 | 正确性无回归，收益跨多轮成立；失败实验也留证 | before/after、方差、PR、rollback | DeepSeek；Codex验收 | 按实验合同 |
| **Advanced Composition** | graph、MTP、multistream、FlagTree、FlagCX、FlagScale、TP/EP/DP独立后再组合 | eager与单变量结果稳定 | ablation能归因；组合无correctness/stability回归；FlagScale不反向成为模型bring-up依赖 | ablation matrix、fallback、稳定性 | DeepSeek；Codex阶段验收 | 仅 multi-node 项需要 |
| **Scale-out Acceptance** | 两机 capacity/communication/生产负载验收（若需要） | 明确触发原因、网络与客户通信约束冻结 | HCCL baseline 和可选 FlagCX 分开验证；SLO、稳定性、故障恢复满足验收 | topology、collective、E2E benchmark、故障记录 | DeepSeek；Codex/用户验收 | **此阶段开始需要第二台 A3 服务器** |

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

前三条任一只有同时满足以下条件才是合法bring-up路径：位于FlagOS Dispatch体系内；无vllm-ascend runtime/package依赖；在910C执行；通过当前correctness microgate；接口支撑GLM-5.2 forward。Reference不是CPU fallback的同义词：tensor可留在NPU并由torch_npu/CANN执行，但必须保存device/backend trace，静默CPU fallback直接判失败。

Eager Correctness前不为了统一算子来源、提高FlagGems覆盖率或追求性能而开发新kernel。Eager Correctness后若profiling证明Reference为瓶颈，再分别比较FlagGems、vendor.ascend、Triton/FlagTree、fusion等单变量优化。

## vLLM-Ascend技术参考与实现边界

- 允许Codex深入研究官方vLLM-Ascend，提取Ascend/910C contract、shape/dtype/layout、KV Cache、prefill/decode、NPU primitive和correctness行为。
- 参考优先级固定为：FlagOS已有跨平台实现 → official vLLM/Transformers → vLLM-Ascend硬件实现参考。
- 生产实现必须进入`vLLM -> vllm-plugin-FL -> FlagOS dispatch/backend -> FlagGems/vendor.ascend/Reference -> torch_npu/CANN` ownership链；不允许把vLLM-Ascend package/runtime带入正式环境。
- 禁止机械复制vLLM-Ascend源码后改名、换目录或重构以隐藏来源。若确有源码复用/派生，必须遵守许可证并保留attribution，不能代码混淆规避声明。
- 每个capability Draft PR必须说明当前缺口、模型contract、hardware primitive、FlagOS ownership/dispatch、correctness test和vLLM-Ascend仅作为reference的具体范围。
- 当前客户边界解释为禁止vLLM-Ascend runtime/package/环境依赖，不扩大为禁止开发阶段研究源码。只有客户以后明确禁止official FlagOS仓库中的历史adapted来源时，才重新做合规判断。

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
