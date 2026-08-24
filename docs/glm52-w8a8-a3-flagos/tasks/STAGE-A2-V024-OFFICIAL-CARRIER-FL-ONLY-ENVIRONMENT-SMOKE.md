# A3-CP-A2-v024 — Official v0.24 Release-Branch Nightly + FlagOS FL-only Environment Smoke

状态：**STOP at Phase A FlagTree gate (2026-08-24T02:52Z)**
执行对象：第一台Ascend A3/910C服务器
第二台服务器：不使用
Evidence目录：`/data/tiankuan/zyg/evidence-a2-v024-20260824T025250Z`

## Phase split

```text
Phase A — Carrier/source/environment preflight without NPU mapping
  -> no NPU device mapping
  -> environment/package/compiler/FL preparation
  -> static distribution/module/entry-point/provider validation
  -> environment gate PASS

Phase B — NPU Runtime Smoke
  -> only after Phase A PASS; no additional confirmation
  -> share only NPU 12+13 while existing tasks continue
  -> create a new NPU-enabled disposable container
  -> replay the validated Phase A environment steps
  -> tiny torch_npu tensor + FlagOS Dispatch operator smoke
  -> STOP for Codex review
```

Phase A不得映射任何NPU device。Phase A全部PASS后可在同一任务中直接进入Phase B；任一环境门禁失败不得接触NPU。

## Fixed identities

- Official documented release tag：`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`；当前无可用artifact，仅保留文档身份
- A2 provisional carrier：`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`
- Carrier identity：仅称official `releases/v0.24.0rc` A3 nightly；不得称为rc1 release image
- Formal project branch：`project/glm52-w8a8-v024`
- FL HEAD：`a9435a34dcd7d0a38e3a853535947371a6c62205`
- FL tree：`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagGems：tag`v5.3.4` / commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9` / tree`87e4e1e98c80dfd31d923bd726795f385aa28ffd`
- FlagTree：`0.6.1rc1+ascend3.5`，仅FlagOS official resource index
- Runtime selectors：`VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`、`VLLM_VENDOR`unset
- Ascend settings：`TRITON_ALL_BLOCKS_PARALLEL=1`、eager

不得从documented release源码、tag名或历史recipe预设provisional carrier内的OS/Python/CANN、torch/torch-npu、vLLM、triton/triton-ascend、transformers或其他runtime tuple；Phase A container preflight记录的实际值才是本次事实。

## Authorized network

只允许：

1. Carrier只接受上述exact digest；本机缺失或RepoDigest不匹配时STOP，禁止使用release tag、mutable nightly tag或其他fallback。
2. Phase A container内从FlagOS official resource index获取exact FlagTree。
3. Phase A container内从official FlagGems获取v5.3.4 exact source。
4. Formal FL source可从唯一repo direct clone；GitHub不可达时允许使用由GitHub可达控制机生成、预置于WORKDIR且附expected SHA256 sidecar的唯一Git bundle relay到Evidence `source/`。

禁止`pip install -U`、其他index/repo/branch、通用联网补包和核心runtime升级。安装使用`--no-deps`；source/editable安装同时使用`--no-build-isolation`。

## Shared Evidence rule

任务启动时冻结`WORKDIR="$(pwd -P)"`。Host Evidence只能写入一个新目录：

`$WORKDIR/a3-cp-a2-v024-env-and-shared-npu-smoke-<hostname>-<UTC timestamp>/`

Phase A与Phase B使用该目录下独立子目录；Phase B不得改写Phase A raw evidence。

不可写即STOP，不得自行切换到`$HOME`、`/tmp`或其他目录。Container writable layer除外。禁止记录credential/token或无关任务数据。

## Phase A — No-NPU Environment Preparation

状态：**Ready**

### Host/container boundary

- 可以只读记录当前16个logical devices均被占用；这不是Phase A STOP条件。
- 检查exact carrier digest不需要NPU。
- 创建全新、名称唯一的disposable container，**不映射任何NPU device**，不执行NPU runtime smoke。
- Formal source只读；package变更只发生在container writable layer。
- 不build/commit新image，不删除已有image/container。
- Phase A结束时停止no-NPU container并保留ID/status；仅在环境gate PASS后新建受限Phase B container。

### A1. Initial inventory

记录image/container identity、OS/Python/CANN package paths，以及torch、torch-npu、vLLM、vllm-ascend、triton/triton-ascend、FlagTree、FlagGems、transformers、compressed-tensors、ATB/NNAL、distributions、entry points和module origins。任何版本只能标为preflight实测，不得用rc1 source tuple补值。

不要求torch_npu发现设备；不得把无NPU设备导致的runtime unavailable判为环境失败。

### A2. Remove vllm-ascend plugin

- 仅用Python package manager卸载`vllm-ascend`。
- 不得连带改变核心runtime。
- 卸载后使用全新的Python process检查：distribution不存在、`find_spec("vllm_ascend") is None`、无有效`vllm_ascend:...`entry point。
- 任一残留即STOP；保存origin/`.pth`/editable path，不手工删除文件。

### A3. Compiler preflight and replacement

- 先记录carrier实际存在的`triton`、`triton-ascend`或其他provider distributions、RECORD、module/native extension和entry points，不预设名称或版本。
- 仅用package manager卸载与FlagTree共享/冲突的实际distribution；不得因历史recipe假定`triton-ascend==3.2.1`。
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
- 不存在时优先从唯一formal repo/branch direct clone。GitHub不可达时，验证relay bundle的expected SHA256后从bundle clone到Phase A Evidence `source/`；禁止解压为普通目录、使用无expected SHA256 archive或切换其他repo/branch。
- 最终必须验证branch=`project/glm52-w8a8-v024`、HEAD=`a9435a34dcd7d0a38e3a853535947371a6c62205`、tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`、worktree clean。
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

Phase A STOP时立即停止。Phase A PASS时记录环境gate结果，然后无需再次确认，按下述边界进入Phase B。

## Phase B — NPU Runtime Smoke

状态：**Ready after Phase A PASS**

只允许共享logical NPU 12+13。进入前及结束后只读记录两者的进程、显存和AICore状态；不得kill、暂停或干扰任何现有任务。可用显存明显下降、AICore持续升高、现有任务异常或无法判断干扰风险时立即STOP。

Phase B必须创建一个新的NPU-enabled disposable container，按Phase A已验证步骤快速重放环境；不得使用`docker commit`或把Phase A container保存成新image。

Phase B范围仅包括：

- container只暴露NPU 12+13及必要Driver/DCMI资源；
- 极小`torch_npu` tensor/device smoke，不执行collective；
- FlagOS Dispatch registry/effective Ascend policy；
- 一个最低风险、小shape、NPU-resident Dispatch operator；
- selected impl origin、输入输出device、结果和无silent CPU fallback。

严格禁止模型加载、vLLM serve、HCCL/TP、KV cache、完整`WorkerFL`/`ModelRunnerFL` runtime、Qwen/GLM、MLA/DSA/Indexer/W8A8、完整attention/MoE、benchmark/profile及大tensor。

### A2 PASS / STOP

只有Phase A全部identity/environment/provider/source门禁和Phase B两项tiny smoke均通过，才报告`A2-v024 PASS`。

下列任一报告`A2-v024 STOP`：carrier digest不匹配；bundle expected SHA256或FL branch/HEAD/tree/clean失败；需要预设或改变核心runtime tuple；FlagTree/FlagGems/FL安装不闭合；provider ownership混合；tiny smoke需要扩大范围；共享NPU状态恶化或可能干扰现有任务。

## Evidence minimum

必须保存：`REPORT.md`（Confirmed/Unknown/Conflict/Potential Blocker）、command manifest、raw stdout/stderr、checksums、exact image digest/container identity、实际runtime inventory、package pre/post diff、vllm-ascend negative check、compiler ownership、FlagGems/FL source/install、bundle SHA256与Git identity、NPU 12+13 pre/post状态、tiny tensor与Dispatch selected impl/device/result，以及明确的A2 PASS/STOP。

---

## 2026-08-24 Execution Results — A2-V024 STOP

**Decision:** STOP at Phase A — FlagTree install gate

**Evidence:** `/data/tiankuan/zyg/evidence-a2-v024-20260824T025250Z`

### PASS

| Gate | Detail |
|---|---|
| Carrier digest | `sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585` exact match |
| FL Git identity | HEAD=`a9435a34dcd7d0a38e3a853535947371a6c62205` MATCH; tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b` MATCH; worktree CLEAN |
| FL source | GitHub direct clone `yanceng305-collab/vllm-plugin-FL-a3-flagos` branch=`project/glm52-w8a8-v024` |
| FlagGems identity | HEAD=`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9` MATCH; tree=`87e4e1e98c80dfd31d923bd726795f385aa28ffd` MATCH |
| vllm-ascend negative | distribution=NOT FOUND; find_spec=NOT FOUND; entry-point=NOT FOUND; import=NOT FOUND (all fresh Python process) |
| Runtime inventory | OS=Ubuntu 22.04.5; Python=3.12.13; CANN=9.0.1; torch=2.10.0+cpu; torch_npu=2.10.0.post2; vllm=0.24.0+empty; triton=3.5.0(pip)/3.2.0(runtime); triton-ascend=3.2.1(uninstalled); transformers=5.13.0 |

### STOP

| Gate | Detail |
|---|---|
| **FlagTree install** | 0.6.1rc1+ascend3.5 cannot be installed for Python 3.12.13 |

**Blocker analysis:**
1. FlagOS official resource index unreachable (HTTP 503)
2. FlagTree source build (tag 0.6.1rc1+ascend3.5) fails: cmake 4.4.0 > <4.0 constraint; missing CUDA NVCC for offline build; nvidia-toolchain-version.json bug (missing `ptxas-blackwell` key)
3. FlagTree 0.6.1+ascend3.5 pre-built from FlagOS Docker image is Python 3.11.15 only; pybind11 Python version mismatch prevents loading in Python 3.12
4. All FlagOS Docker images (harbor.baai.ac.cn/flagos-inner-models-release) use Python 3.11; no Python 3.12 flagtree exists

**Python 3.12 status: Unresolved.** This STOP does NOT conclude "FlagTree does not support Python 3.12." The official FlagTree Python 3.11 pre-built path exists and works. The Python 3.12 blocker is due to the carrier's Python version, not a FlagTree limitation.

### Not Reached

- NPU 12+13 pre/post state (not collected)
- Tiny torch_npu tensor smoke
- FlagOS Dispatch operator smoke
- Phase B container creation

### Next Steps

1. Obtain FlagTree for Python 3.12 (pre-built wheel or buildable source)
2. Or user-approved Python 3.11 carrier variant
3. Re-run A2-v024 with updated FlagTree availability
