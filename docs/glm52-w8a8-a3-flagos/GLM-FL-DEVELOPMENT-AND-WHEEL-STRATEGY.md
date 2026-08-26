# GLM-5.2 FL 开发与 Wheel 交付策略

更新时间：2026-08-26

## 目的

记录后续恢复 GLM-5.2-W8A8 × FlagOS × Ascend A3/910C 适配时的 FL 安装、开发、native operator 编译与最终交付方式，避免后续新会话重新讨论或把开发态安装方式误当成最终交付方式。

本文件记录的是 **User 已确认的未来工程工作流方向**。它不代表当前 GLM 项目已恢复执行，不改变 `D-078` 的 `PAUSED by User Decision` 状态，也不自行改变未来恢复时需要重新确认的 vLLM / FL baseline、Code branch、CANN 或模型 contract。

## 核心结论

后续 GLM 开发采用：

```text
editable-first development
        ↓
按需增量开发 / 编译 vendor.ascend native operators
        ↓
模型功能与 correctness 稳定
        ↓
clean A3 wheel build
        ↓
在不依赖源码 checkout 的环境中安装 wheel
        ↓
standalone FL / model / serve / benchmark validation
```

即：

- **开发态：editable install 优先**，用于快速修改和验证 FL Python / model / Dispatch / Worker / ModelRunner 等代码。
- **native operator：按需编译**。若修改 `vendor.ascend`、C/C++/Ascend native extension、OPP 等二进制部分，仍必须重新编译对应 native artifact；editable 本身不能让已编译二进制自动更新。
- **最终交付态：wheel 优先**。代码稳定后必须从 clean / frozen source 构建 A3 wheel，并以 wheel 安装后的独立 runtime 作为最终 correctness、serve、benchmark 与性能验证基线。

## 为什么开发阶段优先 editable

GLM 后续预计会出现大量快速迭代：

```text
运行 GLM
  ↓
发现 model / Dispatch / backend / operator gap
  ↓
定位当前执行路径
  ↓
修改 Python glue / model integration / Dispatch / backend
  ↓
重新运行验证
```

如果每次 Python 层修改都强制执行完整 wheel build → uninstall/install → rerun，会显著增加开发迭代成本。

使用 editable install 时，Python 层代码通常直接从 working source tree 加载，因此更适合开发和调试。

典型开发态目标：

```text
FL source checkout
  ↓
pip install -e .
  ↓
vllm_fl import / plugin registration
  ↓
PlatformFL
  ↓
WorkerFL
  ↓
ModelRunnerFL
  ↓
FlagOS Dispatch
  ↓
FlagGems / vendor.ascend / NPU-resident Reference
```

具体安装命令、build isolation、依赖策略必须以恢复 GLM 时冻结的实际 branch/build metadata 为准；本文件不冻结未来版本的具体命令。

## editable 不等于“不完整 FL”

`pip install -e .` 与 wheel 安装的主要差异不是功能目标，而是代码加载和交付形态：

| 项目 | Editable install | Wheel install |
|---|---|---|
| `vllm_fl` Python 功能 | 可用 | 可用 |
| PlatformFL / WorkerFL / ModelRunnerFL | 可用 | 可用 |
| FlagOS Dispatch | 可用 | 可用 |
| Python 源码修改后快速生效 | 是 | 否，需要重新 build/install |
| 运行时依赖源码 checkout | 是 | 否 |
| 适合开发调试 | 是 | 一般 |
| 适合最终冻结/交付 | 一般 | 是 |

因此：

> editable 是开发态 FL；wheel 是冻结后的可交付 FL。两者不应被理解成“残缺 FL”和“完整 FL”的区别。

## native operator 的特殊规则

如果 GLM 后续需要补充 Ascend 专用实现，例如：

- MLA / DSA / SFA 相关关键算子；
- Indexer；
- W8A8 Linear / packed quantization 路径；
- MoE / routing；
- KV/cache 相关特殊操作；
- 其他 FlagGems / NPU-resident Reference 无法满足 correctness 或后续性能要求的能力；

并将其实现到 FL `vendor.ascend` 的 native backend，则开发循环应区分：

```text
Python-only 修改
  → editable source 直接验证

native / OPP 修改
  → 重新编译对应 native artifact
  → 再进行运行验证
```

不能因为使用 editable install 就跳过 native extension / OPP 的必要编译。

## 最终为什么仍然要构建 wheel

GLM 开发完成后，最终验收不应永久依赖源码目录的 editable link。

最终需要证明：

```text
frozen clean source
  ↓
A3 / target SoC build
  ↓
FL wheel
  ↓
clean / standalone runtime install
  ↓
无需原始 source checkout 也能 import / load / execute
  ↓
FlagOS ownership 路径成立
  ↓
GLM-5.2-W8A8 correctness
  ↓
serve / benchmark / performance
```

若最终 GLM FL 包含 native Ascend 实现，则 wheel 应同时携带该版本所需的 native extension / OPP 等安装 artifact，并在最终环境中验证实际加载与 NPU 执行。

因此最终工程原则为：

> **开发阶段 editable-first；native op 按需增量编译；功能稳定后 clean wheel build；最终以 wheel 安装环境作为正式 runtime baseline。**

## 与 Qwen3.6 A3 Validation 项目的关系

独立 Qwen3.6-35B-A3B A3 Validation 项目提供了一个有价值的工程参考：当前 Qwen 分支具有明确的 Ascend native build 路径，会构建 `vendor.ascend` native extension 和目标 SoC OPP，并将其打包进 FL wheel。

这说明在存在 FL-owned Ascend native operator 时，wheel 可以成为：

```text
FL Python runtime
+
model / dispatch integration
+
vendor.ascend native extension
+
target-SoC OPP / native artifacts
```

的冻结交付单元。

但不得把 Qwen 项目的版本、branch、具体 custom-op 集合或构建命令机械复制到 GLM。Qwen 的验证结果只能作为工程经验；GLM 恢复后仍需基于其真实 baseline 和模型能力缺口决定哪些算子需要 native implementation。

## 与原 GLM A2 v0.24 clean-room 路线的关系

历史 GLM v0.24 A2 clean-room runtime 使用过 FL editable install。该历史路线中，FL 本身并未像当前 Qwen 分支一样建立完整的 Ascend `_C_ascend + OPP` 打包路径；Ascend 执行可通过 FlagGems / FlagTree / torch_npu / NPU-resident Reference 等合法路径完成。

因此：

- 历史 GLM 没有构建 FL A3 native wheel，并不意味着当时遗漏了一个必做步骤；
- 当前 Qwen 需要 native wheel，是其 branch/build architecture 的具体要求；
- 后续 GLM 是否新增 `vendor.ascend` native operator，应由真实模型 gap 与 backend ownership 决定，而不是为了“形式上和 Qwen 一样”强制新增。

## 客户“原生 FlagOS”要求与安装方式的关系

客户合规性的核心仍遵守 Control repo 现有决策：按**实际模型执行 ownership**判断，而不是按 editable / wheel 这种安装形式判断。

合法目标路径仍是：

```text
vLLM interface
  ↓
vllm-plugin-FL
  ↓
PlatformFL / WorkerFL / ModelRunnerFL
  ↓
FlagOS Dispatch
  ↓
FlagGems / vendor.ascend / NPU-resident Reference
  ↓
FlagTree / torch_npu / CANN
  ↓
Ascend A3 / 910C
```

因此：

- editable install 本身不会使 FL runtime 变得“不原生”；
- wheel install 本身也不能单独证明 FlagOS ownership；
- 最终仍要通过 runtime / backend / operator provenance 证明关键模型执行没有绕过 FlagOS Dispatch 进入不允许的 backend。

## 恢复 GLM 时的建议执行阶段

```text
1. Resume contract / baseline review
2. 建立开发容器与 editable FL
3. GLM model loading / first eager bring-up
4. 按真实 gap 修 Python / Dispatch / backend
5. 必要时实现并增量编译 vendor.ascend native op
6. Eager correctness 稳定
7. Clean source + A3 wheel build
8. 新/干净 runtime 安装 wheel
9. Standalone FL + native artifact validation
10. GLM correctness / serve
11. Baseline benchmark
12. Profile-driven optimization
```

其中第 7～9 步属于从开发态进入正式冻结 runtime 的交付门槛，而不是每一次开发修改都必须重复执行的循环。

## 不由本文件决定的事项

本文件不冻结以下内容：

- 恢复 GLM 时使用 vLLM 0.20.2、0.24 或其他版本；
- 实际 FL upstream / project branch / SHA；
- Python、torch、torch_npu、CANN、FlagTree、FlagGems 的具体版本；
- GLM 最终需要哪些 `vendor.ascend` native operators；
- wheel 的具体文件名和 build command；
- 当前 `PAUSED by User Decision` 状态。

这些内容在 GLM 正式恢复时按当时最新正式仓库、官方 source、模型 contract 与 User Decision 重新冻结。
