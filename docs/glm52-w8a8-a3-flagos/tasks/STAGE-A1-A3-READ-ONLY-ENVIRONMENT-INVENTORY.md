# Stage A1 Task Contract — A3 Read-only Environment Inventory

状态：**Superseded / Dormant；Not Ready；embedded execution prompt MUST NOT be used**

Task ID：`A3-CP-A1`
Parent：`Stage A — Official Carrier FL-only Bring-up`
执行对象：第一台Ascend A3/910C服务器
执行者：DeepSeek
验收者：Codex
第二台服务器：不需要

## 目标

该任务原用于第一次接触A3服务器时冻结现场事实。用户已提供当前Container边界所需Host事实，R0 tuple不再等待Host CANN分析；本任务保留为历史/补证合同，未经新授权不得执行。文件下方旧DeepSeek提示词包含已Superseded的`R0-clean exact tuple`目的和措辞，**不得下发**；未来若需Host补证必须按当前runtime-ownership边界另建任务。

## Evidence目录

任务启动时首先冻结当前工作目录：

```text
WORKDIR="$(pwd -P)"
```

Evidence目录必须新建在该目录下：

```text
$WORKDIR/a3-readonly-inventory-<hostname>-<UTC timestamp>/
```

如果`WORKDIR`不可写，立即停止并报告`blocked`；不得自行切换到`$HOME`、`/tmp`或其他目录。

唯一允许的写入是新建上述Evidence目录，并在其中保存本任务产生的raw logs、命令清单、`REPORT.md`和校验清单。不得覆盖已有目录，不得修改目录外任何文件。

## 必查字段

### Host与基础资源

- OS、发行版、kernel、architecture；
- hostname；
- CPU型号、socket/core/NUMA基本信息；
- 系统内存；
- mount与可用磁盘空间。

### Ascend硬件与占用

- `npu-smi info`或系统中可用的等价只读信息；
- NPU数量、型号、health/status；
- 每个device HBM容量与当前使用量；
- physical package与logical device presentation；
- `/dev/davinci*`等device节点；
- 当前占用NPU的进程；不得输出无关进程的完整argv或可能含credential的参数。

### PyTorch NPU视图

- 仅在现有Python环境已经安装torch与torch-npu、且普通import可安全执行时，记录：
  - torch版本；
  - torch-npu版本；
  - `torch.npu.is_available()`；
  - `torch.npu.device_count()`；
  - 可安全读取的device properties。
- 不得创建venv、安装package、set device、分配测试tensor、执行kernel或触发编译。
- 无法安全执行时标记Unknown并说明原因。

### Driver、Firmware与CANN

- Driver版本与可读版本文件；
- Firmware版本；
- CANN toolkit版本、安装路径、`latest`等symlink实际指向；
- NNAL/ATB/HCCL相关安装路径与可读取版本；
- 当前系统package manager中已有的Ascend/CANN/HCCL/ATB条目；只查询，不修改。

### Python与软件包存在/来源现状

- Python executable与版本；
- pip executable与版本；
- 当前Python distribution和module可见性中是否存在：
  - `vllm`；
  - `vllm-ascend` / `vllm_ascend`；
  - `vllm-plugin-FL` / `vllm_fl`；
  - FlagGems / `flag_gems`；
  - FlagCX；
  - FlagTree；
  - Triton / triton-ascend；
  - torch / torch-npu。
- 对存在项记录version和origin/location；不得import不必要的业务module。

### 环境变量

- 当前登录shell的`PATH`、`PYTHONPATH`、`LD_LIBRARY_PATH`；
- 显式Ascend、CANN、HCCL、NPU、VLLM、FlagOS、Triton相关变量；
- 不得dump完整environment或任何token、password、credential变量；
- 不得source新的`set_env.sh`后把结果冒充当前状态。

### Docker现状

- Docker client/server版本与基本runtime信息；
- 本机已有Docker images：repository、tag、digest、image ID、创建时间/大小（系统能安全提供多少就记录多少）；
- 当前container的ID、image、状态、名称和网络等非敏感字段；
- 识别本机已有official FL A3 carrier或其他CANN/torch-npu candidate image；仅凭名称或tag不能确认runtime ownership，必要信息不足写Unknown；
- 没有candidate image时只记录，不下载。

### HCCL与NPU网络

- 网络接口、地址、路由、link基本信息；
- HCCN工具是否存在；
- 每个可见NPU的只读HCCN/IP信息；
- 可读的HCCN/HCCL配置路径与基础内容；
- 不修改IP、路由、link或任何网络配置。

### 模型目录

- 在当前mount和执行上下文已知/常见模型根目录中，检查是否存在：
  - GLM-5.2-W8A8；
  - Qwen3.6-27B canary；
  - 其他明确相关Qwen canary。
- 只记录目录、总体大小和`config.json`、`quant_model_description.json`、safetensors index等metadata文件是否存在；
- 不遍历整个根文件系统，不读取大权重内容，不计算全量权重hash。

## 严格只读边界

禁止：

- pip、uv、conda install/uninstall；
- apt、yum、dnf、rpm安装、卸载或修改；
- docker pull/build/run/create/rm/rmi；
- 创建或启动container；
- 修改或持久化环境变量；
- 修改Driver、Firmware、CANN、HCCL、网络、路由或device配置；
- 删除、移动、覆盖Evidence目录外文件；
- kill、pkill、killall或改变进程；
- clone、commit、push或修改Git仓库；
- 启动Qwen、GLM、vLLM或任何模型服务；
- 下载image、wheel、模型、源码或外部资料；
- 访问第二台服务器；
- 自行选择R0-clean tuple、搭建环境或进入下一Stage。

命令或工具不存在时，DeepSeek可以选择其他只读方法；不得通过安装工具解决。需要提权才能读取时记录Unknown/Permission denied，不得自行扩大权限。

## Raw evidence要求

- DeepSeek自行选择适合实际系统的只读命令，不要求使用固定shell函数或固定命令编号；
- 每次查询必须保存：实际命令或查询方法、UTC时间、stdout、stderr、exit code；
- 每项raw evidence独立可定位，失败和权限不足同样保留；
- `REPORT.md`中的每个Confirmed结论必须指向对应raw evidence；
- Evidence目录中保存命令清单和文件校验清单；
- 不在raw logs中故意采集完整env、完整进程argv、Docker credential/proxy配置或秘密。

## `REPORT.md`格式

报告必须包含：

1. Executive Summary；
2. Inventory表：字段、观测值、状态、raw evidence；
3. `Confirmed`；
4. `Unknown`；
5. `Conflict`；
6. `Potential Blocker`；
7. 五个必须回答的问题；
8. 命令缺失、失败、权限不足及替代只读方法；
9. 未执行事项声明。

不得把推测写成Confirmed；不同来源不一致时写Conflict；缺少决定R0-clean所需的事实时写Potential Blocker。

## 五个必须回答的问题

1. 真实A3到底向PyTorch暴露多少个NPU device？
2. 官方物理规格`8×128GB`与现场logical device presentation是什么关系？
3. 当前Driver/CANN是否有足够证据支撑拟议clean FlagOS stack？证据不足时必须写Unknown/Potential Blocker，不得自行冻结tuple。
4. Host、当前Python环境或现有Docker images中是否存在vllm-ascend？必须把installed/discoverable与activated/imported/called分开；存在本身不等于违规。
5. 本机是否已有official FL A3 carrier或其他CANN/torch-npu candidate image？没有则只记录，不下载；仅凭image名称不能宣布runtime ownership。

## DeepSeek完整执行提示词

```text
你正在执行项目任务A3-CP-A1：A3 Read-only Environment Inventory。

目标：第一次接触第一台Ascend A3/910C服务器时，只采集并冻结真实环境事实，为Codex后续设计R0-clean exact tuple提供输入。不要搭建环境，不要启动模型，不要做版本决策。

任务启动时立即执行并冻结：

WORKDIR="$(pwd -P)"

Evidence目录必须新建为：

$WORKDIR/a3-readonly-inventory-<hostname>-<UTC timestamp>/

如果WORKDIR不可写，立即停止，状态报告为blocked；不要改用$HOME、/tmp或其他目录。

唯一允许的写入是新建该Evidence目录，并在其中保存raw logs、命令/查询清单、REPORT.md和校验清单。不得覆盖已有目录或修改目录外文件。

你可以根据实际A3操作系统、已安装工具和权限自行选择合适的只读命令。某命令不存在时，可以换用其他只读方法；不得安装工具、修改环境或扩大权限。

必须采集：

1. OS、发行版、kernel、architecture、hostname。
2. CPU、NUMA、系统内存、mount和可用磁盘空间。
3. NPU数量、型号、health/status、每device HBM容量/使用量、physical与logical presentation、device节点、当前NPU占用进程。
4. 如果当前Python已经安全具备torch和torch-npu：torch/torch-npu版本、torch.npu.is_available()、torch.npu.device_count()及安全可读device properties。不得安装、set device、分配tensor、执行kernel或触发编译；不能安全执行则写Unknown。
5. Driver、Firmware、CANN toolkit、NNAL/ATB/HCCL的版本、安装路径、symlink实际指向和现有系统package记录。
6. Python/pip版本，以及当前distribution/module中vllm、vllm-ascend/vllm_ascend、vllm-plugin-FL/vllm_fl、FlagGems、FlagCX、FlagTree、Triton/triton-ascend、torch、torch-npu是否存在；对存在项记录version和origin/location。
7. 当前登录shell的PATH、PYTHONPATH、LD_LIBRARY_PATH及显式Ascend/CANN/HCCL/NPU/VLLM/FlagOS/Triton变量。不得dump完整env或credential变量，不得source新的set_env.sh。
8. Docker client/server版本和基本runtime信息；本机已有images及非敏感container状态；是否存在official FL A3 carrier或其他CANN/torch-npu candidate image。没有则记录，不pull；仅凭名称不能确认runtime ownership。
9. 网络接口、地址、路由、link、HCCN工具、每个可见NPU的只读HCCN/IP信息，以及可读HCCN/HCCL配置。不得修改网络。
10. 在当前mount和已知/常见模型根目录中检查GLM-5.2-W8A8、Qwen3.6-27B及相关Qwen canary是否存在。只记录目录、总体大小和config/quant description/safetensors index等metadata；不要遍历整个根文件系统、读取大权重内容或计算全量权重hash。

严格禁止：

- install/uninstall任何package；
- apt/yum/dnf/rpm修改；
- docker pull/build/run/create/rm/rmi；
- 创建或启动container；
- 修改或持久化环境变量；
- 修改Driver、Firmware、CANN、HCCL、网络、路由或device配置；
- 删除、移动、覆盖Evidence目录外文件；
- kill或改变任何进程；
- clone、commit、push或修改Git仓库；
- 启动Qwen、GLM、vLLM或任何模型服务；
- 下载image、wheel、模型、源码或外部资料；
- 访问第二台服务器；
- 自行决定R0-clean版本组合、搭环境或进入下一Stage。

需要提权才能读取的字段记录为Unknown/Permission denied，不要自行使用提权绕过。

Raw evidence要求：

- 每次查询保存实际命令或方法、UTC时间、stdout、stderr和exit code；
- 失败、命令缺失和权限不足必须保留；
- REPORT.md中的Confirmed必须链接到raw evidence；
- Evidence目录内生成命令清单和文件校验清单；
- 不采集完整进程argv、完整env、Docker credential/proxy信息、token或password。

REPORT.md必须严格包含：

- Executive Summary；
- Inventory表（字段、观测值、状态、raw evidence）；
- Confirmed；
- Unknown；
- Conflict；
- Potential Blocker；
- 命令缺失/失败/权限不足及替代方法；
- 未执行事项声明。

必须直接回答：

1. 真实A3向PyTorch暴露多少个NPU device？
2. 8×128GB官方物理规格与现场logical presentation是什么关系？
3. 当前Driver/CANN是否有足够证据支撑拟议clean FlagOS stack？证据不足必须写Unknown/Potential Blocker，不得决定tuple。
4. Host、当前Python或已有Docker image中是否存在vllm-ascend？把存在性与实际runtime participation分开；存在本身不判违规。
5. 本机是否已有official FL A3 carrier或其他CANN/torch-npu candidate image？没有则只记录，不下载；信息不足时写Unknown。

最终返回：任务状态、WORKDIR绝对路径、Evidence目录绝对路径、REPORT.md完整内容、raw evidence清单、命令失败/权限不足、Potential Blocker和需要Codex决策的事项。

完成后立即停止。不得开始R0-clean环境搭建或任何下一Stage。
```

## Codex验收与下一步

DeepSeek结果返回后，Codex负责：

1. 核对raw evidence与`REPORT.md`结论；
2. 更新Confirmed、Unknown、Conflict、Potential Blocker；
3. 重新判断Driver/Firmware/CANN/Python/torch/torch-npu/compiler profile；
4. 决定R0-clean exact tuple；
5. 只有用户批准后，生成下一条Official Carrier FL-only Environment Smoke执行任务。

DeepSeek不得自行执行第3～5项。
