# DeepSeek执行提示词 — A3-CP-A2-v024

状态：**Superseded by Phase split / Do not execute**

该prompt把free valid pair作为整个A2的前置门禁，现已被Phase A/Phase B拆分取代。当前只允许使用[`DEEPSEEK-A3-CP-A2-V024-PHASE-A-EXECUTION-PROMPT.md`](DEEPSEEK-A3-CP-A2-V024-PHASE-A-EXECUTION-PROMPT.md)。以下内容仅保留历史审计。

执行Ready任务：`Official v0.24 Carrier + FlagOS main FL-only Environment Smoke`。

唯一控制合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`。本提示词与合同冲突时立即STOP并报告，不自行解释扩大权限。

## 目标

在第一台A3/910C服务器上，用official v0.24 A3 carrier创建一次性实验container，在container内完成：

```text
vllm-ascend plugin removal
  -> fresh-process negative check
  -> triton-ascend to exact FlagTree provider replacement
  -> FlagGems v5.3.4
  -> FlagOS new-main FL installation
  -> valid A3 logical-device pair smoke
  -> PlatformFL / WorkerFL / ModelRunnerFL / Dispatch
  -> one small NPU-resident synthetic operator
  -> STOP
```

不要加载Qwen或GLM，不启动模型服务，不进入capability或性能工作。

## 固定版本与source

- Carrier：`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`
- FL formal project branch：`project/glm52-w8a8-v024`
- FL HEAD：`a9435a34dcd7d0a38e3a853535947371a6c62205`
- FL tree：`e5e073edf4b65c053e954d78d20365aab0e1f46b`
- FlagGems：tag`v5.3.4`，commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`，tree`87e4e1e98c80dfd31d923bd726795f385aa28ffd`
- FlagTree：`0.6.1rc1+ascend3.5`；Git tag reference`9a90fddf166d33217777b662821072c41015b294`；wheel仅来自FlagOS official resource index并记录hash
- Carrier initial compiler：`triton-ascend==3.2.1`
- `VLLM_PLUGINS=fl`
- `VLLM_FL_PLATFORM=ascend`
- `VLLM_VENDOR`保持unset
- `TRITON_ALL_BLOCKS_PARALLEL=1`
- eager execution

FlagGems是正式组件，但operator smoke不强制选择FlagGems；以effective `ascend.yaml`和Dispatch实际选择为准。

## 命令与执行风格

根据现场OS、Docker、Python、NPU topology和package状态自行选择合理命令。不要生成或执行固定几十条命令的runner。

命令必须最小、可审计、与当前步骤直接相关。每条命令、退出码、stdout/stderr写入manifest/raw logs。不要输出token、credential、完整敏感环境或其他任务数据。

## Evidence位置

任务开始冻结：`WORKDIR="$(pwd -P)"`。确认可写；不可写即STOP，不得换到`$HOME`、`/tmp`或其他目录。

唯一Host Evidence目录：

`$WORKDIR/a3-cp-a2-v024-fl-only-smoke-<hostname>-<UTC timestamp>/`

除一次性container writable layer外，Host侧只允许写该目录。

## Host preflight

Official rule：`A3 requires at least 2 NPUs to work together`。

在创建container前：

1. 重新检查OC2和全部NPU进程占用；不得kill、暂停或干扰。
2. 用DCMI/npu-smi/topology等只读事实确定至少两个logical devices组成的valid A3 working pair。
3. 不假设一定是`0+1`、`2+3`；优先同physical-card pair，但必须有现场映射证明。
4. 两个device必须都明确空闲；映射不明、pair不完整或任一占用即STOP。
5. 检查exact carrier是否已存在；存在则记录RepoDigest/image ID。
6. 若不存在，只允许pull唯一指定的official carrier tag。禁止其他tag、其他image或自动fallback；pull后记录identity。
7. 确认formal repo的project branch、HEAD/tree正确且Host working tree clean；后续只读挂载。

## 网络权限

仅允许：

- pull唯一指定carrier；
- 从FlagOS official resource index获取exact FlagTree；
- 从official `flagos-ai/FlagGems`获取exact v5.3.4 source。
- 若服务器缺少formal FL source，只允许从`yanceng305-collab/vllm-plugin-FL-a3-flagos` clone唯一branch `project/glm52-w8a8-v024`，建议放在Evidence `source/`。

禁止`pip install -U`、依赖自动升级、其他index/tag/image或通用联网补包。安装优先`--no-deps`；source/editable安装必须加`--no-build-isolation`。如果exact artifact无法获取或要求改变torch、torch-npu、vLLM、CANN、transformers、compressed-tensors等核心runtime，STOP。

## Disposable container

创建全新、名称唯一的实验container，只暴露选定valid pair和必要Driver/DCMI资源。Formal repo只读挂载。所有package变更只限container writable layer。

不得build/commit新image，不修改原始image，不删除已有image/container。任务结束后停止任务container并保留ID/status供审查。

## 执行阶段

### A. 初始inventory

冻结image/container identity、device visibility/topology，以及Python、CANN、torch、torch-npu、vLLM、vllm-ascend、triton/triton-ascend、FlagTree、FlagGems、transformers、compressed-tensors、ATB/NNAL、entry points和module origins。保存核心package freeze。

### B. 移除vllm-ascend plugin

仅用Python package manager卸载`vllm-ascend`。如果会连带改变其他核心runtime，STOP。

卸载后启动一个全新的Python process检查：

- distribution不存在；
- `importlib.util.find_spec("vllm_ascend")`为`None`；
- 无有效`vllm_ascend:...` plugin entry point。

任一失败即STOP。记录残留origin、`.pth`或editable path；禁止手工删除文件制造PASS。

### C. Compiler replacement

先记录carrier中`triton`/`triton-ascend`distribution、RECORD、`triton.__file__`、`triton._C`、backend/entry point与native origin。

仅用package manager卸载distribution `triton`（若存在）和`triton-ascend`。禁止手工`rm`site-packages、`.so`、`.pth`或dist-info。

仅从FlagOS official resource index安装`flagtree==0.6.1rc1+ascend3.5`，禁止依赖升级。

安装后证明：

- distribution inventory可解释；
- `triton.__file__`、`triton.__version__`明确；
- `triton._C` origin明确；
- Ascend backend/entry point和active driver/provider明确；
- RECORD/file ownership一致；
- 不存在明显混合的triton-ascend与FlagTree文件树；
- 最终只有一个single coherent FlagTree-owned provider。

无法证明即STOP，不手工清理修补。

### D. FlagGems v5.3.4

已有安装必须能证明exact tag/commit，否则从official repo获取v5.3.4。保留`.git`，验证HEAD/tree/clean后，使用当前container Python执行等价：

`python -m pip install --no-build-isolation --no-deps <FlagGems-source>`

明确禁止执行：

- `setup.sh`；
- `flaggems-setup`；
- 任何会创建独立venv/Python的bootstrap；
- 任何自动安装backend dependencies/compiler的bootstrap流程。

如果当前container不满足FlagGems build requirement，STOP；不得运行setup/bootstrap绕过，也不得联网安装其ascend-cann900 profile。不得升级runtime。

### E. FL new-main

服务器已有formal repo且exact project branch存在时，保持该repo readonly。

若服务器不存在formal repo，允许从唯一GitHub repo `yanceng305-collab/vllm-plugin-FL-a3-flagos` clone唯一branch `project/glm52-w8a8-v024`，建议clone到Evidence目录下`source/`。不得clone其他branch/repo。Clone后必须验证exact HEAD、tree和clean；不匹配即STOP。

将`project/glm52-w8a8-v024`完整复制到container writable staging，保留`.git`，安装前验证exact HEAD/tree/clean。

只允许等价：

`python -m pip install --no-build-isolation --no-deps -e <writable-staging>`

`_version.py`、egg-info和build artifacts只留staging，不回写Host。缺build requirement且需要联网补包或改变核心runtime时STOP。

### F. Minimal smoke

1. 再次确认valid pair仍完整空闲。
2. 验证torch/torch-npu只使用该pair，完成低风险NPU tensor/device smoke。
3. 验证实际`PlatformFL`、`WorkerFL`、`ModelRunnerFL`class/module origin。
4. 验证FlagOS Dispatch registry及effective Ascend policy。
5. 只执行一个最低风险、小shape、NPU-resident synthetic operator；优先`rms_norm`或`silu_and_mul`之一。
6. 保存Dispatch selected impl、callable origin、输入输出device和执行结果；证明无silent CPU fallback。

不为强制触发FlagGems或Triton扩大范围。

## 严格禁止

- 第二台服务器；
- 修改Host Driver/Firmware/CANN/HCCN/HCCL/network；
- kill或暂停现有任务；
- 修改control或正式代码repo、commit、push、PR；
- Qwen/GLM权重或模型服务；
- MLA、DSA/SFA、Indexer、W8A8、完整attention/MoE；
- benchmark/profile/性能优化；
- 手工删除package文件；
- FlagGems `setup.sh`、`flaggems-setup`或任何bootstrap/独立环境流程；
- 未授权image/tag/index/package；
- 自行进入下一Stage。

## Evidence要求

至少保存：

- `REPORT.md`，分Confirmed / Unknown / Conflict / Potential Blocker；
- command manifest、raw stdout/stderr、checksums；
- image/container identity；
- valid pair topology/occupancy；
- package pre/post diff与核心runtime未变化；
- vllm-ascend fresh-process negative check；
- compiler distribution/RECORD/module/native/driver/provider ownership；
- FlagGems与FL SHA/tree/clean/install；
- 若使用formal FL clone fallback，记录唯一repo、branch、remote、HEAD/tree/clean；
- Platform/Worker/ModelRunner/Dispatch origins；
- synthetic op、selected impl、tensor device与结果。

## PASS / STOP

全部合同满足才报告`A2-v024 PASS`。下列任一情况报告`A2-v024 STOP`并立即停止：

- 无完整free valid pair或占用变化；
- exact carrier/FlagTree/FlagGems/FL无法获取或验证；
- vllm-ascend negative check失败；
- compiler无法形成single coherent FlagTree provider；
- package transaction改变核心runtime；
- FL/FlagGems staging SHA/tree/clean失败；
- 缺build requirement且需额外联网；
- synthetic smoke要求模型、capability代码或范围扩张；
- 发生任何禁止行为。

最终回复Codex：状态、Evidence绝对路径、image/container identity、device pair、package diff、negative check、provider ownership、FlagGems/FL identity、FL classes/Dispatch、synthetic result、四类结论、偏差与禁止行为确认。

完成A2后立即停止，不进入canary或GLM Stage。
