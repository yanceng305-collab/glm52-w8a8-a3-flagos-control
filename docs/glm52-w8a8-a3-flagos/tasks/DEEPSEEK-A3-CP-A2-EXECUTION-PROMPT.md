# PAUSED — DeepSeek执行提示词 — A3-CP-A2 v0.20.2 Historical Smoke

状态：**Superseded / Paused by upstream branch migration / DO NOT DISTRIBUTE OR EXECUTE**

本提示词基于official `v0.2.1@92a6f767...`、vLLM0.20.2和carrier`v0.20.2rc1-a3`。它没有设计new main/vLLM0.24、FlagGems v5.3.4、FlagTree rc1或A3 valid logical-device pair，因此不得交给服务器。以下内容仅保留历史审计，不构成当前授权。

历史原文（不得执行）曾将本任务描述为Ready：`A3-CP-A2 — Official Carrier FL-only Environment Smoke`。

这是第一台Ascend A3/910C服务器上的受控环境smoke。目标仅是从本机已有official A3 carrier创建一次性实验container，在container内移除`vllm-ascend` plugin，然后证明基础Ascend软件栈与最小FlagOS执行链仍可工作。

不要把本任务解释为“vllm-ascend image/package存在即违规”。official carrier与package coexistence路线仍然合法；本次卸载只是为了降低FL-only首次bring-up变量。完整coexistence/dynamic provenance审计已经后置，本任务不得扩张到该范围。

## 一、唯一事实源和固定输入

执行必须遵守控制仓库中的A2 task contract：

`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`

固定输入：

- Carrier：`quay.io/ascend/vllm-ascend:v0.20.2rc1-a3`
- 正式代码仓库：`yanceng305-collab/vllm-plugin-FL-a3-flagos`
- FL baseline commit：`92a6f7670465922c60e88f06787b8f0923e761f3`
- FL baseline tree：`e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`
- Runtime selector：`VLLM_PLUGINS=fl`
- FL platform selector：`VLLM_FL_PLATFORM=ascend`
- `VLLM_VENDOR`必须保持unset
- FlagGems是preferred而非mandatory；不得要求所有operator都走FlagGems

不得修改控制面、正式代码仓库或任何GitHub仓库。不得自行进入下一Stage。

## 二、任务目标

只验证以下结果：

1. 本机已有的exact official carrier能够启动；
2. 只在新建的一次性实验container内卸载`vllm-ascend` distribution；
3. 卸载后由新Python process完成三项minimal negative check；
4. Python、CANN、torch、torch-npu、vLLM及NPU基础能力未被破坏；
5. `PlatformFL`、`WorkerFL`、`ModelRunnerFL`与FlagOS Dispatch最小闭环成立；
6. 一个最低风险、小shape、NPU-resident synthetic operator通过FlagOS Dispatch选择合法实现并成功执行。

本任务不是模型验证。不要加载任何Qwen或GLM权重，不要启动正式vLLM模型服务。

## 三、命令选择原则

你可以根据A3实际OS、Docker、Python和package布局自行选择合理的检查与验证命令。不要机械照搬固定runner，也不要为了输出更多信息扩大测试范围。

所有命令必须：

- 与本任务目标直接相关；
- 尽量只读、低开销；
- 避免输出credential、token、完整敏感环境或其他任务数据；
- 在失败时保留原始stdout/stderr与退出码；
- 记录到commands/query manifest，保证Codex可以复核。

## 四、Evidence目录

任务开始时首先冻结当前工作目录：

`WORKDIR="$(pwd -P)"`

确认该目录可写。不可写则立即STOP，不得自行切换到`$HOME`、`/tmp`或其他目录。

在WORKDIR下新建唯一Evidence目录：

`$WORKDIR/a3-cp-a2-official-carrier-fl-only-smoke-<hostname>-<UTC timestamp>/`

除一次性container自身的可写层外，所有本任务产生的Host侧日志、报告、manifest和checksum只能写入该Evidence目录。

Evidence至少包含：

- `REPORT.md`
- 原始日志
- commands/query manifest
- checksum manifest
- carrier image和container identity
- pre/post package inventory
- minimal negative check结果
- FL source/staging校验证据
- Platform/Worker/ModelRunner/Dispatch证据
- NPU-resident synthetic operator smoke证据

`REPORT.md`必须明确分为：`Confirmed`、`Unknown`、`Conflict`、`Potential Blocker`。

## 五、执行前强制preflight

先执行低开销只读检查，不创建container，直到以下条件全部明确：

1. 重新检查第一台A3上当前OC2 benchmark及其他NPU进程占用；
2. 只选择明确空闲的最小logical device范围；不得使用已占用或状态不确定的device；
3. 检查本机是否已经存在exact carrier tag；记录实际RepoDigest、image ID、architecture和创建信息；
4. 确认正式FL repo在本机可读，HEAD/tree与冻结baseline一致；后续只能readonly挂载；
5. 确认WORKDIR和Evidence目录规则满足。

以下任一成立必须立即STOP，且不得创建container：

- 没有明确空闲logical device；
- device占用无法可靠判断；
- carrier image本机不存在；
- carrier RepoDigest或image ID无法确认；
- 正式FL repo缺失、HEAD/tree不匹配或存在会影响副本验证的工作树变化；
- 必要的container device/Driver挂载边界无法确定。

如果image不存在或digest不确定，只报告。禁止`docker pull`。

## 六、严格安全边界

禁止：

- kill、暂停或干扰任何现有任务；
- 使用已占用或不确定的NPU device；
- 操作第二台服务器；
- `docker pull`、`docker build`或commit新image；
- 删除现有image或container；
- 修改Host Driver、Firmware、CANN、HCCN、HCCL或network；
- 修改Host持久环境变量或系统配置；
- 修改正式A3代码repo、checkout其他SHA、创建commit、push或PR；
- 安装任何GLM capability patch；
- 加载Qwen或GLM权重；
- 启动vLLM模型服务；
- 进入MLA、DSA/SFA、Indexer、W8A8、完整MoE或attention验证；
- benchmark、profile或性能优化；
- 完整`sys.modules`生命周期、loaded `.so`、native library、call-site或coexistence provenance trace。

不要自行提权。遇到权限或挂载问题时记录并STOP。

## 七、一次性实验container

只有preflight全部通过后，才创建一个全新、名称唯一的一次性实验container：

- 来源必须是已确认identity的本机official carrier；
- 只暴露已选定的明确空闲最小logical device；
- 只挂载必要的NPU/Driver/DCMI资源；
- 正式FL repo只能readonly挂载；
- 不修改原始image；
- 不创建新image；
- 记录container ID、名称、image ID、device visibility和关键非敏感环境选择器。

任务结束时让该任务container停止，但保留其ID和状态供Codex审查；不要自行删除。

## 八、container初始inventory

在任何package变更前，记录实际版本与origin：

- Python
- CANN及相关路径
- torch
- torch-npu
- vLLM
- vllm-plugin-FL / `vllm_fl`
- FlagGems
- FlagTree
- triton / triton-ascend
- vllm-ascend / `vllm_ascend`
- vLLM platform/general plugin entry points

发现vllm-ascend存在是official carrier的预期事实，不得据此判FAIL。

## 九、允许的package变更

只允许在本任务新建container的可写层内执行标准Python package卸载，目标仅为`vllm-ascend` distribution。

在卸载前先检查package manager给出的目标和依赖影响。如果卸载会同时删除、降级或修改CANN、torch、torch-npu、vLLM、Triton、ATB/NNAL、FlagGems或其他非目标runtime组件，立即STOP，不要继续。

不要卸载、重装或升级任何其他carrier package。

## 十、卸载后的minimal negative check

卸载完成后，必须启动一个全新的Python process。该process不能是pre-uninstall inventory所用解释器，也不能复用其module cache或entry-point对象。

只检查：

1. `vllm-ascend` Python distribution不存在；
2. `importlib.util.find_spec("vllm_ascend")`返回`None`；
3. vLLM plugin entry points中不再存在实际指向`vllm_ascend:...`的有效entry point。

任何一项失败都立即STOP。保存distribution位置、module origin、`.pth`/editable path、entry-point来源与原始输出。

不得为了PASS而直接删除源码、`.so`、`.pth`、entry-point metadata或site-packages内容。

## 十一、冻结FL baseline的安装规则

先inventory container中已有的vllm-plugin-FL来源和版本。如果它已经能够明确对应冻结baseline，可以继续使用并保存证明。

如果FL缺失或不能证明等于冻结baseline：

1. 正式FL repo继续readonly；
2. 将完整repo复制到container内部的disposable writable staging目录；
3. 必须保留完整`.git`信息；
4. 在任何安装或build动作前，重新验证staging：
   - HEAD等于`92a6f7670465922c60e88f06787b8f0923e761f3`；
   - HEAD tree等于`e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`；
   - working tree与冻结baseline一致且clean；
5. 只从该writable staging执行`--no-deps` editable安装；
6. 不得修改源码内容或checkout其他版本。

`setuptools_scm`生成的`vllm_fl/_version.py`、egg-info、build metadata及其他artifact只能出现在container staging副本。不得写入或复制回正式repo。

如果`.git`没有正确保留、HEAD/tree不匹配、安装前worktree不clean，或构建试图写回正式repo，立即STOP。

## 十二、基础Ascend栈复核

卸载及必要的FL安装完成后，在第一次NPU操作前再次确认所选logical device仍为空闲；若OC2或其他任务占用发生变化，立即STOP。随后确认：

- Python、CANN、torch、torch-npu、vLLM和compiler相关package仍可正常inventory/import；
- torch_npu能够看见且只使用所选空闲device；
- 能在该device完成最低风险的NPU tensor/device smoke；
- 没有静默CPU fallback；
- 未发生非授权package版本变化。

不要对compiler/provider做深度trace。只记录Triton相关distribution、FlagTree/triton-ascend是否存在及import origin。只有后续synthetic smoke自然触发Triton时，才顺手记录active provider；不要专门扩大测试。

## 十三、FlagOS最小ownership验证

在不加载模型、不启动服务的前提下确认并保存module/class origin：

1. effective selector为`VLLM_PLUGINS=fl`；
2. current platform为`vllm_fl.platform.PlatformFL`；
3. effective worker为`vllm_fl.worker.worker.WorkerFL`；
4. effective model runner为`vllm_fl.worker.model_runner.ModelRunnerFL`；
5. FlagOS Dispatch registry/policy正常加载；
6. effective Ascend per-op policy可以读取。

这些必须是实际环境中的有效选择结果，不能只引用源码字符串或README。

## 十四、唯一synthetic operator smoke

只执行一个最低风险、小shape、NPU-resident synthetic operator smoke。

优先在`rms_norm`与`silu_and_mul`中选择当前环境最容易通过现有公开接口安全构造的一项。先审查现有FL测试、operator签名和dispatch调用方式，再选择；不要同时执行两项来凑覆盖率。

必须证明并保存：

- 调用经过FlagOS Dispatch；
- selected implementation属于FlagGems、`vendor.ascend`或Reference之一；
- callable/module origin可识别；
- 输入和输出tensor均保持在NPU；
- 执行成功；
- 不存在静默CPU fallback。

若第一个候选因必要接口前置无法在不扩大范围的情况下安全构造，可以改用另一个候选，并记录更换原因。若两者都需要模型权重、服务、完整attention/MoE或代码修改，STOP，不得扩大范围。

## 十五、PASS与STOP

只有以下全部成立才能报告`A2 PASS`：

- 本机existing exact official carrier成功启动；
- 只在任务container内卸载vllm-ascend；
- 新Python process中的三项negative check全部通过；
- 若安装FL，writable staging通过exact HEAD/tree/clean校验，生成artifact只存在副本，正式repo零修改；
- carrier基础Ascend stack未破坏；
- NPU可用且只使用明确空闲device；
- PlatformFL、WorkerFL、ModelRunnerFL确认；
- FlagOS Dispatch确认；
- 一个NPU-resident synthetic operator经FlagOS ownership成功执行；
- Evidence完整；
- 所有禁止边界均未触发。

遇到task contract中的任何停止条件时，报告`A2 STOP`，保留已经获得的只读证据，不进行补救性安装、手工清理或范围扩张。

A2 PASS不代表Qwen、GLM、W8A8、MLA、DSA/SFA、Indexer、完整attention/MoE、compiler provenance或vllm-ascend coexistence已经验证。

## 十六、最终报告格式

完成或STOP后，回复Codex：

1. 状态：`A2 PASS`或`A2 STOP`；
2. Evidence绝对路径与`REPORT.md`路径；
3. hostname、carrier RepoDigest/image ID、container ID、所选logical device；
4. pre/post package变化摘要；
5. fresh-process三项negative check结果；
6. FL source/staging HEAD、tree、clean status及artifact位置；如未安装说明原因；
7. torch/torch-npu/NPU结果；
8. PlatformFL、WorkerFL、ModelRunnerFL、Dispatch证据；
9. synthetic operator、selected impl、tensor device和结果；
10. Confirmed / Unknown / Conflict / Potential Blocker；
11. 相对task contract的任何偏差；
12. 明确确认是否发生以下行为：pull/build image、修改Host、kill任务、访问第二台、加载模型、修改正式repo、benchmark/profile、进入capability开发。

完成A2后立即停止。不要启动Qwen、GLM或下一Stage，等待Codex审查与验收。
