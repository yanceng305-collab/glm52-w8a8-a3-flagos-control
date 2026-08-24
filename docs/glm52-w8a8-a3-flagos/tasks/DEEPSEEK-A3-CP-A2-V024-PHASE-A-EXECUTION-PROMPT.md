# DeepSeek执行提示词 — A3-CP-A2-v024 Phase A

执行Ready任务：`Phase A — No-NPU Environment Preparation`。

唯一合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`。

## 目标

在第一台A3服务器上，不映射任何NPU device，完成v0.24 carrier的FL-only环境准备：

```text
exact carrier acquire/start without NPU mapping
  -> package inventory
  -> uninstall vllm-ascend
  -> fresh-process negative check
  -> triton-ascend to FlagTree static provider replacement
  -> FlagGems v5.3.4 install
  -> FL project branch install
  -> static distribution/module/entry-point/provider validation
  -> STOP and save Evidence
```

当前16个logical NPU全部占用不阻止Phase A。不要等待free pair，也不要创建NPU-enabled container。Phase A结束后必须STOP，不进入Phase B。

## 固定identity

- Carrier：`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`
- Formal repo：`yanceng305-collab/vllm-plugin-FL-a3-flagos`
- Branch：`project/glm52-w8a8-v024`
- FL HEAD：`a9435a34dcd7d0a38e3a853535947371a6c62205`
- FL tree：`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagGems：v5.3.4 / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9` / tree`87e4e1e98c80dfd31d923bd726795f385aa28ffd`
- FlagTree：`0.6.1rc1+ascend3.5`，仅FlagOS official resource index
- Initial compiler：`triton-ascend==3.2.1`
- `VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`、`VLLM_VENDOR`unset
- `TRITON_ALL_BLOCKS_PARALLEL=1`、eager

## 命令风格

根据现场环境自行选择最小、可审计命令，不编写固定几十条命令的runner。记录每条命令、退出码和raw stdout/stderr。不得输出credential、token或无关任务数据。

## Evidence

任务启动冻结`WORKDIR="$(pwd -P)"`。不可写即STOP，不得换目录。

唯一Evidence目录：

`$WORKDIR/a3-cp-a2-v024-phase-a-no-npu-<hostname>-<UTC timestamp>/`

Host侧只写该目录；container writable layer除外。

至少保存：`REPORT.md`、command manifest、raw logs、checksums、image/container identity、package pre/post diff、negative check、compiler ownership、FlagGems/FL source/install和static origin evidence。

`REPORT.md`分Confirmed / Unknown / Conflict / Potential Blocker。

## 允许的网络

仅允许：

1. 本机缺exact carrier时pull唯一指定tag；
2. 从FlagOS official resource index获取exact FlagTree；
3. 从official FlagGems repo获取v5.3.4 exact source；
4. 服务器缺formal FL source时，从唯一formal repo clone唯一project branch到Evidence `source/`。

禁止其他image/tag/index/repo/branch、`pip install -U`、通用依赖补齐或核心runtime升级。

## Phase A container

创建全新、名称唯一的disposable container，**不映射任何NPU device**。不要求free pair，不做torch_npu device smoke，不初始化Ascend worker runtime。

不build或commit新image，不修改原始image，不删除已有image/container。Phase A结束时停止container并保留ID/status。

## 执行步骤

### 1. Carrier与初始inventory

- 本机有exact carrier则记录RepoDigest/image ID；没有则只pull该exact tag并记录identity。
- 创建no-NPU container。
- 记录OS/Python/CANN package paths，以及torch、torch-npu、vLLM、vllm-ascend、triton/triton-ascend、FlagTree、FlagGems、transformers、compressed-tensors、ATB/NNAL、distributions、entry points和module origins。
- 无NPU device导致的runtime unavailable不是Phase A失败。

### 2. vllm-ascend removal

仅用Python package manager卸载`vllm-ascend`，不得改变其他核心runtime。

卸载后启动全新Python process检查：

- distribution不存在；
- `importlib.util.find_spec("vllm_ascend") is None`；
- 无有效`vllm_ascend:...`entry point。

任一残留即STOP；记录origin、`.pth`、editable path。禁止手工删除文件制造PASS。

### 3. Compiler replacement

- 记录初始triton/triton-ascend distributions、RECORD、module/native extension和entry points。
- 仅用package manager卸载distribution`triton`（若存在）和`triton-ascend`。
- 从FlagOS official resource index安装exact`flagtree==0.6.1rc1+ascend3.5`，使用`--no-deps`，禁止runtime升级。
- Post-install记录distribution inventory、`triton.__file__`、`triton.__version__`、`triton._C`origin、Ascend backend/entry point及RECORD/file ownership。
- 只做provider静态验证，不要求active NPU driver初始化。
- 必须形成single coherent FlagTree-owned文件树，不得存在明显triton-ascend/FlagTree混合ownership；否则STOP。
- 禁止手工rm package文件或dist-info。

### 4. FlagGems v5.3.4

- 获取/确认exact source，保留`.git`并验证HEAD、tree、clean。
- 只用当前container Python执行等价：

`python -m pip install --no-build-isolation --no-deps <FlagGems-source>`

禁止`setup.sh`、`flaggems-setup`和任何创建独立venv/Python、自动安装backend dependencies/compiler的bootstrap。

缺build requirement时STOP，不运行bootstrap绕过或联网补backend profile。

### 5. FL project source/install

服务器已有exact formal project branch时readonly使用。

若不存在，只允许从`yanceng305-collab/vllm-plugin-FL-a3-flagos` clone branch`project/glm52-w8a8-v024`到Evidence `source/`。禁止其他repo/branch。

验证HEAD=`a9435a34dcd7d0a38e3a853535947371a6c62205`、tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`、working tree clean。

完整复制到container writable staging并保留`.git`，再次验证identity，只允许等价：

`python -m pip install --no-build-isolation --no-deps -e <writable-staging>`

Build artifacts只留staging，不回写Host。缺build requirement或需改变核心runtime时STOP。

### 6. Static validation

在不初始化NPU的范围内验证并记录：

- package pre/post diff与核心runtime版本未变化；
- FL、FlagGems、FlagTree distributions/modules/origins；
- FL platform/general plugin entry points；
- `PlatformFL`、`WorkerFL`、`ModelRunnerFL`class/module静态origin；
- Dispatch package/config/module静态可见性；
- FlagTree Ascend backend/provider静态origin。

若某个import会触发NPU初始化，不强行执行，记录Unknown；留给Phase B。

## 禁止

- 映射任何NPU device；
- 创建NPU-enabled container；
- 执行torch_npu device smoke、Worker/ModelRunner实际Ascend初始化、Dispatch runtime或synthetic NPU operator；
- 进入Phase B，即使期间出现free pair；
- `docker commit`或build新image；
- 第二台服务器；
- kill/暂停现有任务；
- 修改Host Driver/Firmware/CANN/network；
- Qwen/GLM、MLA/DSA/Indexer/W8A8、完整attention/MoE；
- benchmark/profile；
- FlagGems setup/bootstrap；
- 手工删除package文件；
- 修改control或正式代码repo、commit/push/PR。

## PASS / STOP

只有environment、negative check、single coherent static provider、FlagGems、FL和静态验证全部完成，才报告`A2-v024 Phase A PASS`。

下列任一报告`A2-v024 Phase A STOP`：

- exact依赖/source无法获取或验证；
- vllm-ascend negative check失败；
- compiler文件树混合或ownership不明；
- package transaction改变核心runtime；
- SHA/tree/clean失败；
- 缺build requirement需额外联网；
- 正式repo发生写入；
- 需要NPU才能继续Phase A。

最终回复Codex：状态、Evidence绝对路径、image/container identity、package diff、negative check、compiler static ownership、FlagGems/FL identity、static origins、四类结论、偏差与禁止行为确认。

Phase A完成后立即停止。不要创建NPU container，不进入Phase B或任何下一Stage。
