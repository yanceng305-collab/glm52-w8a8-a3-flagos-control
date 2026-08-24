# A3-CP-A2-v024 — Official v0.24 Carrier + FlagOS main FL-only Environment Smoke

状态：**Phase A Ready / Phase B Waiting for valid free pair**
执行对象：第一台Ascend A3/910C服务器
第二台服务器：不使用

## Phase split

```text
Phase A — No-NPU Environment Preparation
  -> no NPU device mapping
  -> environment/package/compiler/FL preparation
  -> static distribution/module/entry-point/provider validation
  -> STOP and save Evidence

Phase B — NPU Runtime Smoke
  -> only after a complete valid free A3 pair exists
  -> create a new NPU-enabled disposable container
  -> replay the validated Phase A environment steps
  -> NPU/Platform/Worker/ModelRunner/Dispatch/synthetic-op smoke
  -> STOP for Codex review
```

没有free pair不阻止Phase A。Phase A不得映射任何NPU device，不得进入Phase B。Phase B将另行生成执行提示词。

## Fixed identities

- Carrier：`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`
- Formal project branch：`project/glm52-w8a8-v024`
- FL HEAD：`a9435a34dcd7d0a38e3a853535947371a6c62205`
- FL tree：`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagGems：tag`v5.3.4` / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9` / tree`87e4e1e98c80dfd31d923bd726795f385aa28ffd`
- FlagTree：`0.6.1rc1+ascend3.5`，仅FlagOS official resource index
- Carrier compiler starting state：`triton-ascend==3.2.1`
- Runtime selectors：`VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`、`VLLM_VENDOR`unset
- Ascend settings：`TRITON_ALL_BLOCKS_PARALLEL=1`、eager

## Authorized network

只允许：

1. 本机缺exact carrier时pull上述唯一tag；禁止其他image/tag/fallback。
2. Phase A container内从FlagOS official resource index获取exact FlagTree。
3. Phase A container内从official FlagGems获取v5.3.4 exact source。
4. 服务器缺formal FL source时，从唯一repo`yanceng305-collab/vllm-plugin-FL-a3-flagos` clone唯一branch`project/glm52-w8a8-v024`到Evidence `source/`。

禁止`pip install -U`、其他index/repo/branch、通用联网补包和核心runtime升级。安装使用`--no-deps`；source/editable安装同时使用`--no-build-isolation`。

## Shared Evidence rule

每个Phase启动时冻结`WORKDIR="$(pwd -P)"`。Host Evidence只能写入：

- Phase A：`$WORKDIR/a3-cp-a2-v024-phase-a-no-npu-<hostname>-<UTC timestamp>/`
- Phase B：未来另建独立目录，不复用或修改Phase A Evidence。

不可写即STOP，不得自行切换到`$HOME`、`/tmp`或其他目录。Container writable layer除外。禁止记录credential/token或无关任务数据。

## Phase A — No-NPU Environment Preparation

状态：**Ready**

### Host/container boundary

- 可以只读记录当前16个logical devices均被占用；这不是Phase A STOP条件。
- Pull或检查exact carrier identity不需要free pair。
- 创建全新、名称唯一的disposable container，**不映射任何NPU device**，不执行NPU runtime smoke。
- Formal source只读；package变更只发生在container writable layer。
- 不build/commit新image，不删除已有image/container。
- Phase A结束时停止container并保留ID/status，不进入Phase B。

### A1. Initial inventory

记录image/container identity、OS/Python/CANN package paths，以及torch、torch-npu、vLLM、vllm-ascend、triton/triton-ascend、FlagTree、FlagGems、transformers、compressed-tensors、ATB/NNAL、distributions、entry points和module origins。

不要求torch_npu发现设备；不得把无NPU设备导致的runtime unavailable判为环境失败。

### A2. Remove vllm-ascend plugin

- 仅用Python package manager卸载`vllm-ascend`。
- 不得连带改变核心runtime。
- 卸载后使用全新的Python process检查：distribution不存在、`find_spec("vllm_ascend") is None`、无有效`vllm_ascend:...`entry point。
- 任一残留即STOP；保存origin/`.pth`/editable path，不手工删除文件。

### A3. Compiler replacement

- 记录初始`triton`/`triton-ascend`distributions、RECORD、module/native extension和entry points。
- 仅用package manager卸载distribution`triton`（若存在）和`triton-ascend`。
- 从FlagOS official resource index安装exact`flagtree==0.6.1rc1+ascend3.5`，使用`--no-deps`，禁止runtime升级。
- Post-install记录distribution inventory、`triton.__file__`、`triton.__version__`、`triton._C`origin、Ascend backend/entry point和RECORD/file ownership。
- Phase A只做provider静态验证，不要求NPU active driver初始化。
- 必须证明文件树由一个coherent FlagTree provider拥有，不存在明显triton-ascend/FlagTree混合ownership；否则STOP。
- 禁止手工`rm`site-packages、`.so`、`.pth`或dist-info。

### A4. FlagGems v5.3.4

- 获取/确认exact source，保留`.git`并验证commit/tree/clean。
- 只允许当前container Python执行等价`python -m pip install --no-build-isolation --no-deps <FlagGems-source>`。
- 禁止`setup.sh`、`flaggems-setup`及任何创建独立venv/Python、自动安装backend dependencies/compiler的bootstrap流程。
- 缺build requirement时STOP，不运行bootstrap或联网补backend profile。

### A5. FL project installation

- 服务器已有exact project branch时readonly使用。
- 不存在时只允许clone唯一formal repo/branch到Phase A Evidence `source/`，随后验证exact HEAD/tree/clean；禁止其他repo/branch。
- 完整复制到container writable staging并保留`.git`。
- 安装前再次验证HEAD/tree/clean。
- 只允许等价`python -m pip install --no-build-isolation --no-deps -e <writable-staging>`。
- `_version.py`、egg-info和build artifacts只留staging，不回写Host。
- 缺build requirement或需改变核心runtime时STOP。

### A6. Static validation

在不初始化NPU的范围内记录：

- Python distribution/module/entry-point inventory；
- FL、FlagGems、FlagTree版本与origin；
- FL platform/general plugin entry points；
- `PlatformFL`、`WorkerFL`、`ModelRunnerFL`class/module静态origin；
- Dispatch package/config/module静态可见性；
- FlagTree Ascend backend/provider静态origin；
- package pre/post diff和核心runtime版本未变化。

若某项安全import会触发NPU初始化，则不强行import，记录Unknown并保留证据；Phase B负责runtime验证。

### Phase A PASS / STOP

全部环境、package、compiler、FlagGems、FL和静态验证通过且Evidence完整，报告`A2-v024 Phase A PASS`。

下列任一报告`A2-v024 Phase A STOP`：

- exact carrier/FlagTree/FlagGems/FL无法获取或验证；
- vllm-ascend negative check失败；
- compiler文件树混合或ownership不明；
- package transaction改变核心runtime；
- source SHA/tree/clean失败；
- 缺build requirement需额外联网；
- 正式repo发生写入；
- 需要NPU才能继续Phase A范围。

Phase A PASS或STOP后都立即停止，不创建NPU-enabled container，不进入Phase B。

## Phase B — NPU Runtime Smoke

状态：**Waiting for a complete valid free A3 pair / No prompt yet**

只有第一台A3出现经只读topology证明的完整、明确空闲valid pair后才可执行。不得硬编码`0+1`、`2+3`等编号，不kill或暂停OC2。

Phase B必须创建一个新的NPU-enabled disposable container，按Phase A已验证步骤快速重放环境；不得使用`docker commit`或把Phase A container保存成新image。

Phase B范围仅包括：

- torch/torch-npu valid-pair smoke；
- `PlatformFL`、`WorkerFL`、`ModelRunnerFL`实际Ascend初始化；
- FlagOS Dispatch和effective Ascend policy；
- 一个最低风险small-shape NPU-resident synthetic operator；
- selected impl origin、NPU input/output和无silent CPU fallback。

Phase B仍不加载Qwen/GLM，不进入MLA/DSA/Indexer/W8A8、完整attention/MoE或benchmark/profile。

## Evidence minimum

Phase A必须保存：`REPORT.md`（Confirmed/Unknown/Conflict/Potential Blocker）、command manifest、raw stdout/stderr、checksums、image/container identity、package pre/post diff、vllm-ascend negative check、compiler ownership、FlagGems/FL source/install、static origin evidence，以及明确的Phase A PASS/STOP。

Phase B未来使用独立Evidence并引用Phase A报告/checksum，不修改Phase A目录。
