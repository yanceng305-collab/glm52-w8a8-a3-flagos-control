# A3-CP-A2 — Official Carrier FL-only Environment Smoke

状态：**Ready**
Parent：`Stage A — Official Carrier FL-only Bring-up`
执行授权：用户已批准本task进入Ready并生成DeepSeek执行提示词；本轮Codex不代为操作服务器
执行对象：第一台Ascend A3/910C服务器
第二台A3：不需要

## 工程意图

以official FL Ascend Dockerfile使用的A3 image作为环境carrier，在一次性实验container内移除`vllm-ascend` plugin，低成本验证其余Ascend软件栈与FlagOS最小执行链能否独立工作。该操作只是为首次bring-up减少变量，**不是**把“vllm-ascend image/package存在”重新定义为违规，也不否定official package coexistence路线。

本任务只验证环境和synthetic operator，不加载Qwen或GLM权重，不启动正式vLLM模型服务。

## 固定输入

- Carrier：`quay.io/ascend/vllm-ascend:v0.20.2rc1-a3`。
- 正式FL代码仓库：`yanceng305-collab/vllm-plugin-FL-a3-flagos`。
- 冻结baseline：commit `92a6f7670465922c60e88f06787b8f0923e761f3`，tree `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`。
- Runtime selectors：`VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`；`VLLM_VENDOR`保持unset。
- FlagGems为preferred而非mandatory；operator按effective `ascend.yaml`在FlagGems、`vendor.ascend`、Reference之间选择。

## Ready gate

执行前必须同时满足：

1. **Satisfied：** 用户已批准A3-CP-A2 Ready task与DeepSeek执行提示词。
2. 任务启动时冻结`WORKDIR="$(pwd -P)"`；该目录可写。不可写则停止，不得自行切换到`$HOME`、`/tmp`或其他目录。
3. 重新检查当前NPU进程与logical-device占用；只允许选择明确空闲的最小device范围。任何device状态不确定即停止。
4. 本机已有carrier的tag、RepoDigest和image ID可确认。若image不存在或digest不确定，只报告，不得`docker pull`。
5. Host device/Driver/DCMI必要挂载边界已知；不得修改Host Driver、Firmware、CANN、HCCN/HCCL或network。
6. 正式A3代码仓库仍位于冻结baseline，只读挂载且不得被任务修改；container内writable staging方案能够保留完整`.git` metadata并在安装前复核HEAD/tree/clean worktree。
7. 新建唯一Evidence目录：`$WORKDIR/a3-cp-a2-official-carrier-fl-only-smoke-<hostname>-<UTC timestamp>/`。

## 唯一允许的环境变更

冻结baseline的`pyproject.toml`使用`[tool.setuptools_scm] write_to = "vllm_fl/_version.py"`，而source baseline没有预生成该文件。因此editable build需要可写source tree；不得直接对readonly正式repo安装并把由此产生的写失败误判为FlagOS问题。

只允许在本任务新建的一次性实验container内部：

1. 使用标准Python package操作卸载`vllm-ascend` distribution。
2. 如果inventory证明container中的`vllm-plugin-FL`缺失或不等于冻结baseline，先把只读挂载的正式代码仓库完整复制到一次性container内部的writable staging目录；副本必须保留`.git`信息。安装前重新验证副本：HEAD等于`92a6f7670465922c60e88f06787b8f0923e761f3`、HEAD tree等于`e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`、working tree内容与冻结baseline一致且无预存修改。只允许从该disposable writable copy执行`--no-deps` editable安装。

正式repo始终readonly，禁止checkout、生成文件或回写。`vllm_fl/_version.py`、egg-info、build metadata及其他editable/build artifacts只允许出现在container内writable staging副本；不得复制回Host正式repo。

禁止卸载、重装或升级CANN、torch、torch-npu、vLLM、triton/triton-ascend、ATB/NNAL、FlagGems及其他carrier Ascend runtime依赖。不得修改原始image，不得build或commit新image。任务container退出后保留其ID和状态供审查，不自行删除。

## 执行顺序与证据门禁

### 1. Host与carrier preflight

- 冻结hostname、UTC时间、WORKDIR与Evidence路径；
- 记录当前NPU占用、选定空闲logical device及选择理由；
- 记录carrier RepoDigest、image ID、创建时间、architecture和本机存在状态；
- 若image缺失、digest不确定或没有明确空闲device，停止且不创建container。

### 2. Container创建前inventory

- 创建全新、一次性实验container，仅挂载必要NPU/Driver设备和只读代码路径；
- 记录container ID、image ID、环境变量和device visibility；
- inventory Python、CANN、torch、torch-npu、vLLM、vllm-plugin-FL、FlagGems、FlagTree、triton/triton-ascend、vllm-ascend及其origin/version；
- 不因发现vllm-ascend package而判FAIL，这是official carrier的预期事实。

### 3. FL-only变更与最小negative check

在container内卸载`vllm-ascend`后，必须启动一个**新的Python process**执行以下三项检查。该process不得复用pre-uninstall inventory期间的解释器、module cache或已加载entry-point对象：

1. Python distribution `vllm-ascend`不存在；
2. `importlib.util.find_spec("vllm_ascend")`返回`None`；
3. vLLM plugin entry points中不存在仍实际指向`vllm_ascend:...`的有效entry point。

若任一失败，立即停止。保存distribution位置、module origin、`.pth`/editable path和entry-point来源；不得直接删除源码、`.so`、`.pth`或site-packages内容来制造PASS。

A2明确不执行完整`sys.modules`生命周期、loaded `.so`、native library、call-site或coexistence runtime trace；这些属于Post-Eager Runtime Provenance Audit。

### 4. Carrier基础栈复核

卸载后确认原有Python、CANN、torch、torch-npu、vLLM及compiler/provider相关package仍可inventory/import；确认torch_npu能看见所选NPU并完成低风险device/tensor smoke。该步骤不得修改剩余软件tuple。

### 5. FL最小ownership

在不加载模型权重、不启动服务的条件下，记录并确认：

- effective plugin selector为`VLLM_PLUGINS=fl`；
- current platform class/module为`vllm_fl.platform.PlatformFL`；
- effective worker class/module为`vllm_fl.worker.worker.WorkerFL`；
- effective model runner class/module为`vllm_fl.worker.model_runner.ModelRunnerFL`；
- FlagOS Dispatch registry/policy加载成功，effective Ascend per-op policy可读取；
- 至少一个小shape、NPU-resident synthetic operator经FlagOS Dispatch选择FlagGems、`vendor.ascend`或Reference之一并成功执行；tensor输入输出保持在NPU，无静默CPU fallback。

优先从`rms_norm`或`silu_and_mul`中选择一个最低风险、现有接口可直接调用的代表算子。一个算子足以满足A2；只有第一个候选因接口前置无法安全构造时才尝试另一个，不为凑数量扩大范围。

### 6. Compiler/provider轻量inventory

- 只记录Triton相关distribution、FlagTree/triton-ascend是否存在及import origin；
- synthetic smoke若自然触发Triton，可顺手记录active provider；
- 若未自然触发，不专门扩展kernel或trace来识别provider。

## Evidence

所有Host侧任务输出只写入本任务Evidence目录；container自身package变更仅发生在其可写层。Evidence至少包括：

- `REPORT.md`；
- raw logs；
- commands/query manifest；
- checksum manifest；
- image/container identity；
- pre/post package inventory；
- FL readonly source与container writable staging的HEAD、tree、pre-install clean status，以及生成artifact位置；
- minimal negative check结果；
- FL platform/worker/runner/dispatch证据；
- NPU-resident synthetic operator smoke证据；
- Confirmed / Unknown / Conflict / Potential Blocker分类。

不得把敏感环境变量、token、credential或不必要的其他任务数据写入Evidence。

## PASS条件

A2只有在以下全部成立时PASS：

1. official carrier由本机已有exact image成功启动；
2. 只在实验container内成功卸载vllm-ascend；
3. 三项minimal negative check全部通过；
4. negative check明确由卸载后的新Python process执行；
5. 若安装FL，writable staging在安装前通过exact HEAD/tree/clean校验，所有生成artifact仅位于该副本；正式repo保持readonly且零修改；
6. carrier基础Ascend stack未被破坏，torch/torch-npu/NPU可用；
7. `PlatformFL`、`WorkerFL`、`ModelRunnerFL`来源确认；
8. FlagOS Dispatch registry/policy确认；
9. 至少一个NPU-resident synthetic operator经FlagOS ownership成功执行；
10. Evidence完整且未越过安全边界。

PASS不代表Qwen、GLM、W8A8、MLA/DSA/Indexer、attention prefill/decode、compiler完整provenance或vllm-ascend coexistence已验证。

## 停止条件

- carrier image本机不存在或digest不能确认；
- 无明确空闲logical device，或OC2/其他任务占用发生变化；
- 卸载动作将移除或改动CANN、torch、torch-npu、vLLM、Triton/ATB/NNAL等非目标组件；
- minimal negative check残留distribution/module/entry point；
- 必须手工删除源码、`.so`、`.pth`或site-packages才能继续；
- FL baseline无法在不改代码、不改main的条件下使用；
- writable staging未保留`.git`、HEAD/tree不匹配、安装前worktree不clean，或构建产物试图写回正式repo；
- synthetic smoke需要模型权重、服务启动、attention/MoE完整链或capability实现；
- 需要pull/build image、修改Host或访问第二台服务器。

## 明确非目标

- Qwen/GLM权重加载或模型服务；
- MLA、DSA/SFA、Indexer、W8A8、完整MoE或attention验证；
- benchmark、profile、性能优化；
- 完整compiler/provider provenance；
- vllm-ascend coexistence动态审计；
- 正式A3代码仓库修改；
- 第二台服务器。

## A2之后

A2 PASS后立即停止并由Codex验收。下一步是Qwen canary或control批准的等价最小canary；不得自行扩张A2或进入GLM capability实现。完整dynamic provenance按[`POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`](POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md)在GLM-5.2-W8A8 Eager Correctness通过后执行。

对应执行提示词：[`DEEPSEEK-A3-CP-A2-EXECUTION-PROMPT.md`](DEEPSEEK-A3-CP-A2-EXECUTION-PROMPT.md)。
