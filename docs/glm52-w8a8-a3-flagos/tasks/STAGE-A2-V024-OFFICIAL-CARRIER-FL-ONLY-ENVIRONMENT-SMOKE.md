# A3-CP-A2-v024 — Official v0.24 Carrier + FlagOS main FL-only Environment Smoke

状态：**Ready**
执行对象：第一台Ascend A3/910C服务器
第二台服务器：不使用
目标：完成v0.24基础环境smoke；不加载任何模型

## Fixed identities

- Carrier：`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`
- Official FL freeze：`a9435a34dcd7d0a38e3a853535947371a6c62205`
- Official FL tree：`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- Formal project branch：`project/glm52-w8a8-v024`
- Formal immutable baseline：`baseline/official-main-vllm0.24-20260821-a9435a3`
- FlagGems：tag`v5.3.4` / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9` / tree`87e4e1e98c80dfd31d923bd726795f385aa28ffd`
- FlagTree：`0.6.1rc1+ascend3.5`，Git tag reference`9a90fddf166d33217777b662821072c41015b294`；wheel仅使用FlagOS official resource index并记录artifact hash
- Carrier compiler starting state：`triton-ascend==3.2.1`
- vLLM：0.24.0 empty build
- Runtime selectors：`VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`、`VLLM_VENDOR`unset
- Ascend settings：`TRITON_ALL_BLOCKS_PARALLEL=1`、eager

## Authorized network boundary

只允许：

1. 本机不存在exact carrier时，pull上述唯一tag；禁止其他image/tag和自动fallback。
2. Container内从FlagOS official resource index获取exact FlagTree版本。
3. Container内从official `flagos-ai/FlagGems`获取tag v5.3.4，并验证exact commit/tree。
4. 若服务器不存在formal FL source，允许只从`yanceng305-collab/vllm-plugin-FL-a3-flagos` clone唯一branch `project/glm52-w8a8-v024`，建议放入Evidence目录的`source/`；禁止其他repo/branch。

禁止`pip install -U`、通用依赖补齐和resolver升级核心runtime。所有目标安装使用`--no-deps`；source/editable build同时使用`--no-build-isolation`。精确依赖无法获取、需要换tag/index或改变torch、torch-npu、vLLM、CANN、transformers、compressed-tensors等核心版本时STOP。

## Host preflight and device rule

Official rule：`A3 requires at least 2 NPUs to work together`。

1. 任务启动时冻结`WORKDIR="$(pwd -P)"`，Evidence只能写入其下新目录；不可写即STOP。
2. 重新检查OC2和所有NPU占用；不kill、不暂停、不干扰。
3. 使用DCMI/npu-smi/Host topology等只读事实确定至少两个logical devices组成的valid A3 working pair。
4. 不硬编码`0+1`、`2+3`等编号；优先使用同physical-card pair，但必须由现场映射证明。
5. pair必须完整且两个device都明确空闲；映射不确定、任一占用或没有完整free pair即STOP。
6. 记录16 logical devices到8 physical cards的实际映射证据和选pair理由，不重新争论已冻结的16×64GB/1024GB Host事实。

## Disposable container boundary

- 只从exact carrier创建名称唯一的一次性container。
- 只暴露选定valid pair和必要Driver/DCMI设备。
- 正式代码repo以readonly方式挂载。
- 所有package变更只发生在container writable layer。
- 不修改或commit原始image，不build新image。
- 不删除现有image/container；任务container结束后停止并保留identity供审查。

## Required transaction

### 1. Initial inventory

记录image RepoDigest/ID、container ID、OS/Python/CANN、torch/torch-npu、vLLM、vllm-ascend、triton/triton-ascend、FlagTree、FlagGems、transformers、compressed-tensors、ATB/NNAL、entry points、module origins和核心package freeze。

### 2. Remove vllm-ascend plugin

- 仅用Python package manager卸载`vllm-ascend` distribution。
- 卸载不得连带改变其他核心runtime。
- 卸载后用一个全新的Python process检查：distribution不存在、`find_spec("vllm_ascend") is None`、无有效`vllm_ascend:...` plugin entry point。
- 任一残留即STOP；保存origin、`.pth`、editable path与entry-point来源，不手工删除文件。

### 3. Compiler replacement transaction

- 记录carrier初始`triton`/`triton-ascend` distributions、RECORD、module/native extension和entry points。
- 仅用package manager卸载distribution `triton`（若存在）和`triton-ascend`；不得手工`rm` site-packages、`.so`、`.pth`或dist-info。
- 通过FlagOS official resource index安装exact `flagtree==0.6.1rc1+ascend3.5`，禁止依赖升级。
- Post-install必须记录：distribution inventory、`triton.__file__`、`triton.__version__`、`triton._C` origin、Ascend backend/entry point、active driver/provider、RECORD/file ownership。
- 必须证明形成一个single coherent FlagTree-owned `triton` provider，不存在明显混合的triton-ascend/FlagTree文件树。
- 任一provider origin不明、两个distribution冲突、混合ownership或核心package变化即STOP。

### 4. FlagGems v5.3.4

- 若container中已有FlagGems，先验证是否精确对应tag/commit；不能证明则不用该副本。
- 必要时从official FlagGems获取v5.3.4，保留`.git`并验证commit/tree/clean。
- 必须使用当前container Python执行等价`python -m pip install --no-build-isolation --no-deps <FlagGems-source>`；不得升级runtime。
- 明确禁止运行`setup.sh`、`flaggems-setup`或任何会创建独立venv/Python、自动安装backend dependencies/compiler的bootstrap流程。
- 当前container缺少FlagGems build requirement时STOP；不得运行bootstrap绕过，也不得联网补装backend profile。
- FlagGems是正式组件但synthetic operator不强制走FlagGems。

### 5. FL new-main installation

- 优先使用服务器已有formal repo的`project/glm52-w8a8-v024`，Host repo始终readonly。
- 若服务器不存在该repo，允许从唯一GitHub repo `yanceng305-collab/vllm-plugin-FL-a3-flagos` clone唯一branch `project/glm52-w8a8-v024`到Evidence `source/`；clone后必须验证exact HEAD/tree/clean。不得clone其他branch/repo。
- 完整复制到container writable staging并保留`.git`。
- 安装前验证HEAD=`a9435a34dcd7d0a38e3a853535947371a6c62205`、tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`、working tree clean。
- 只允许等价`python -m pip install --no-build-isolation --no-deps -e <writable-staging>`。
- `_version.py`、egg-info和build artifacts只留container staging，不回写Host。
- 缺build requirement或需要改变核心runtime时STOP，不联网补包。

## Minimal validation

1. 再次检查pair仍完整空闲。
2. torch/torch-npu只看见并使用该valid pair；完成低风险NPU tensor/device smoke。
3. 确认`PlatformFL`、`WorkerFL`、`ModelRunnerFL`的实际class/module origin。
4. 确认FlagOS Dispatch registry和effective Ascend policy加载。
5. 只执行一个最低风险、小shape、NPU-resident synthetic operator；优先在`rms_norm`或`silu_and_mul`中选一个。
6. 证明调用经过Dispatch、selected impl来源为FlagGems/`vendor.ascend`/Reference之一、输入输出留在NPU且无silent CPU fallback。
7. 不为强制触发FlagGems或Triton扩大范围。

## Explicitly out of scope

- Qwen/GLM权重或vLLM模型服务；
- MLA、DSA/SFA、Indexer、W8A8、完整attention、完整MoE；
- benchmark/profile/性能优化；
- 第二台服务器；
- Host Driver/Firmware/CANN/network修改；
- control/code repo修改、commit、push或PR；
- 完整coexistence provenance audit；
- 手工删除package文件制造PASS。
- FlagGems `setup.sh`、`flaggems-setup`或其他bootstrap/独立环境流程。

## Evidence

Evidence目录：`$WORKDIR/a3-cp-a2-v024-fl-only-smoke-<hostname>-<UTC timestamp>/`。

至少保存：

- `REPORT.md`，分Confirmed / Unknown / Conflict / Potential Blocker；
- command/query manifest与raw stdout/stderr；
- checksum manifest；
- image/container identity；
- device pair/topology与occupancy证据；
- package pre/post diff和核心runtime不变证据；
- vllm-ascend fresh-process negative check；
- compiler distribution/RECORD/module/native/driver/provider证据；
- FlagGems与FL source SHA/tree/clean/install evidence；
- 若使用formal FL clone fallback，保存唯一repo/branch/remote/HEAD/tree/clean证据；
- Platform/Worker/ModelRunner/Dispatch origin；
- synthetic operator selected impl、tensor device和结果。

除container writable layer外，Host侧任务写入只允许该Evidence目录。不得记录credential、token或无关任务数据。

## PASS / STOP

只有所有步骤通过且无越权，才报告`A2-v024 PASS`。任何以下情况报告`A2-v024 STOP`并保留已有证据：

- exact image无法获得或identity不明；
- 无valid完整free pair或OC2占用变化；
- vllm-ascend negative check失败；
- compiler不能形成single coherent FlagTree provider；
- exact FlagTree/FlagGems/FL无法获取或验证；
- 安装需要升级核心runtime或缺失build requirement；
- 正式repo发生写入；
- synthetic smoke需要模型、capability实现或范围扩张。

A2结束后立即停止，不进入canary或GLM Stage，等待Codex验收。
