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
- GitHub 是唯一项目事实源；新仓库确认前，本文件只是 external-state candidate。

## 当前关键路径

```text
Research Freeze
  -> Clean Provenance (R0-clean，无 vllm-ascend)
  -> 910C Canary (Qwen3.6-27B TP2 eager)
  -> GLM Contract Gate (vLLM语义 + quant format)
  -> Capability Microgates (MLA/DSA/Indexer/W8A8)
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
| **Clean Provenance** | 从中性 CANN base 零起建立 `R0-clean` | 用户授权服务器；base policy、driver/CANN compatibility、repo 已确认 | vllm-ascend package/module/entry point/native lib 全部不存在；FL platform、torch-npu、HCCL、FlagGems/Triton 最小链路通过；fresh rebuild 可复现 | inventory、negative audit、import/native trace、原始日志、环境 hash | DeepSeek 执行；Codex 验收 | 不需要 |
| **910C Canary** | 用官方当前 910C CI-backed 模型隔离验证基础链 | Clean Provenance accepted；Qwen3.6-27B 权重就绪 | TP2 eager offline + serving 正确；FL/dense attention/HCCL dispatch 可追溯；无污染依赖 | prompts/outputs、tolerance、dispatch trace、峰值内存 | DeepSeek；Codex验收 | 不需要 |
| **GLM Contract Gate** | 决定 vLLM 语义基线与 W8A8 artifact contract | Canary accepted；真实 checkpoint manifest 齐全 | ADR 选择 `0.23/0.24 uplift` 或 `0.20.2 backport`；ADR 选择 `AscendV1 native loader` 或经证明等价的 `compressed-tensors conversion` | API/worker diff、IndexShare ownership、tensor/scale/layout 清单 | Codex 决策；DeepSeek仅做授权 spike | 不需要 |
| **Capability Microgates** | 全模型前逐个验证 sparse MLA、DSA/SFA、Indexer、W8A8 Linear/MoE | 两项 contract ADR 批准 | 每项有最小可重复 correctness test、backend selection 与 failure signature；Missing 转为最小实现边界 | dtype/shape/layout、operator结果、日志 | DeepSeek；Codex review | 不需要 |
| **Capacity & Placement** | 基于真实 artifact 与现场 16 logical-device topology 确定布局 | manifest、目标上下文/并发、microgates通过 | per-rank weight/scale/workspace/KV预算；首次 load 的 TP/EP 候选与安全余量冻结 | placement simulation、memory budget、topology | DeepSeek测算；Codex验收 | 默认不需要；不足则触发 |
| **First Eager Load** | 首次 capacity-valid GLM-5.2-W8A8 load，收敛首个真实故障 | placement accepted；已知硬缺口已处理或有明确预期 | 模型构造/权重 load 成功，或只保留一个可复现 first failure；关闭 MTP/graph/FlagCX/multistream | exact run config、first-error log、weight-key audit、dispatch trace | DeepSeek；Codex定下一任务 | 仅 Capacity 明确触发；开始前必须通知 |
| **Minimal Compatibility** | 一次修复一个已证实缺口 | First Eager Load 单一 failure | 最小 patch、focused test、Qwen canary 无回归、Draft PR | diff、tests、provenance、rollback | DeepSeek；Codex PR review | 沿用批准布局 |
| **Eager Correctness** | 完成 capacity-valid eager 正确性 | load/forward blockers关闭 | 输出精度、IndexShare owner/shared、MoE、quant scales、稳定多轮满足标准 | golden/tolerance、layer trace、memory/KV | DeepSeek；Codex阶段验收 | 仅容量需要 |
| **Baseline Benchmark** | 建立未优化基线 | Eager Correctness accepted | 固定输入分布/上下文/并发；给 TTFT、TPOT、吞吐、延迟分位、利用率、功耗、内存和方差 | raw results、env SHA、重复运行 | DeepSeek | 单机先 |
| **Profile & Bottleneck** | 证明首要瓶颈 | baseline稳定 | kernel/communication/host/调度归因；每项有可证伪假设 | profiler trace、timeline、算子/通信占比 | DeepSeek；Codex审查 | 仅证据指向 scale-out 时触发 |
| **Single-variable Optimize** | 每次只改变一个算子或配置 | profile假设批准 | 正确性无回归，收益跨多轮成立；失败实验也留证 | before/after、方差、PR、rollback | DeepSeek；Codex验收 | 按实验合同 |
| **Advanced Composition** | graph、MTP、multistream、FlagTree、FlagCX、TP/EP/DP 独立后再组合 | eager与单变量结果稳定 | ablation 能归因；组合无 correctness/stability 回归 | ablation matrix、fallback、稳定性 | DeepSeek；Codex阶段验收 | 仅 multi-node 项需要 |
| **Scale-out Acceptance** | 两机 capacity/communication/生产负载验收（若需要） | 明确触发原因、网络与客户通信约束冻结 | HCCL baseline 和可选 FlagCX 分开验证；SLO、稳定性、故障恢复满足验收 | topology、collective、E2E benchmark、故障记录 | DeepSeek；Codex/用户验收 | **此阶段开始需要第二台 A3 服务器** |

## 阶段治理

- 每个实现/实验 Stage 使用独立 branch 和 Draft PR；不得自动 merge 或跨 Stage。
- DeepSeek 只能执行控制面中的 `ready` task，不能修改 PLAN/STATUS/DECISIONS，不能自行进入下一 Stage。
- Codex 负责证据审查、技术决策、PR review、阶段验收与控制面更新。
- 任一新事实推翻路线时，保留已验证成果并回退到相应 Contract/Capability/Capacity gate，不全量重启。

## 两机触发规则

以下任一成立才允许申请第二台服务器：

1. 真实 checkpoint 的 per-rank placement + KV/workspace 安全余量证明单机不足；
2. 已进入明确的 HCCL/FlagCX、EP/DP 或 scale-out 对比实验；
3. 客户验收负载不能由单机覆盖。

任务 Ready 前必须直接写明：`此阶段开始需要第二台 A3 服务器`，并给用途、拓扑、通信 backend、停止条件和验收证据。
