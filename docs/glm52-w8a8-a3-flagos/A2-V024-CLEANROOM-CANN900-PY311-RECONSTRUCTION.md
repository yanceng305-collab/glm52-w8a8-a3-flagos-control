# A2 clean-room container/environment reconstruction

状态：**Factual baseline；actual Docker/runtime replay parameters pending final verification**

目标：当现有container丢失时，使新的Codex2能够从frozen base和可审计artifact重建当前A2 runtime。本文只记录正式仓库已确认的事实；实际`docker run`和缺失artifact replay字段必须由final verification从现场metadata/Evidence恢复，不得猜测。

## Frozen identity

| 项目 | 正式值 |
|---|---|
| Base tag | `swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:9.0.0-a3-ubuntu22.04-py3.11-devel` |
| Frozen RepoDigest | `swr.cn-south-1.myhuaweicloud.com/ascendhub/cann@sha256:5f20011b2c5509ca4716393e66fc7aa07016629bce36a7f6c32c1bf31f30433f` |
| Current formal container | `flagos-cann900-py311-test` |
| Original task/run | `A2-V024-CLEANROOM-CANN900-PY311` / `20260824T080753Z` |
| Final verification task | [`A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION`](tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION.md) |

User-confirmed pull方式：

```bash
docker pull swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:9.0.0-a3-ubuntu22.04-py3.11-devel
```

重建时必须在创建container前验证local RepoDigest精确包含上述frozen digest；mutable tag本身不足以替代digest。

## Docker run / entry status

当前Control资料**不能推出实际docker run命令**。Final verification必须从live `docker inspect`、image metadata与既有Evidence恢复一个checksum-bound的`rebuild-container.sh`；本文后续只接收有Evidence pointer的值。

| 参数组 | 当前状态 | Final verification必须恢复 |
|---|---|---|
| Image/container ID | Pending live capture | image ID、container ID、created time、configured image、RepoDigest |
| Device | Pending live capture | 全部host→container device mapping、permissions、device requests/runtime |
| Volumes/mounts | Pending live capture | bind/named volume、source/destination、RW、propagation及driver/runtime/repo/work/artifact路径 |
| Network | Pending live capture | network mode、attached networks、ports、aliases、DNS/extra-hosts（若非默认） |
| SHM / IPC / PID | Pending live capture | `ShmSize`、IPC/PID mode |
| Privilege/security | Pending live capture | privileged、cap add/drop、security opt、cgroup/device rules |
| Process/resources | Pending live capture | user、workdir、hostname、entrypoint/cmd、TTY/stdin、restart、ulimit、CPU/memory limits |
| Environment | Pending live capture | runtime-critical env和bootstrap；secret value只脱敏记录来源，不写入Control |
| Other non-defaults | Pending live capture | inspect中影响重建的其余非默认HostConfig/Config值 |

标准进入候选为：

```bash
docker exec -it flagos-cann900-py311-test /bin/bash
```

其shell、user、workdir及进入后必须source的environment尚待现场确认；final verification必须返回实际验证过的进入方式。不要把本候选当作已确认container创建参数。

## Final runtime tuple

以下为run `20260824T080753Z`报告并由Control索引的最终tuple；final verification只核对，不修改：

| Component | Version / identity |
|---|---|
| Python | `3.11.15` |
| CANN | `9.0.0` |
| torch | `2.10.0+cpu` |
| torch_npu | `2.10.0.post2` |
| vLLM | `0.24.0` |
| FlagTree / Triton API | `0.6.1+ascend3.5` / `3.5.1` |
| FlagGems | `5.3.4` |
| FL | `project/glm52-w8a8-v024@a9435a34dcd7d0a38e3a853535947371a6c62205` / tree `e5e073edf4b65c053e954d78d20365aab0e1f46b` |

最终provider形态：`triton`与`triton-ascend`独立distribution absent；active `triton`namespace由FlagTree拥有。该结论必须在重建后重新验证，不能靠安装名推断。

## Sources and replay inputs

### torch / torch_npu

- Final versions：`torch==2.10.0+cpu`、`torch_npu==2.10.0.post2`，CPython 3.11 / aarch64。
- Original install使用`pip --no-deps`以保持CANN 9.0.0兼容tuple。
- Exact wheel filename、URL、SHA256和retained artifact path：**Pending final verification recovery from supplemental/Work manifests**。

### vLLM empty

- Official source tag/commit：`v0.24.0` / `ee0da84ab9e04ac7610e28580af62c365e898389`。
- Source tarball SHA256：`286571b02164190fd6c5bf86410c7643d5eac8a7dff5d868641b92969c4c2f60`。
- Generated wheel SHA256：`a27c5fbd87c9249b7a6e3b240502f8dc1f6b861a9688c01a4cdf2e9f3f995657`。
- Replay entry：

```bash
VLLM_TARGET_DEVICE=empty VLLM_VERSION_OVERRIDE=0.24.0 \
python -m pip install --no-deps --no-build-isolation .
```

`empty`提供vLLM Python/interface层，不构建device extension；它只满足本A2 tiny Dispatch gate，不能外推为模型级backend/E2E能力。

### FlagTree

- Official index：`https://resource.flagos.net/repository/flagos-pypi-hosted/simple`
- Wheel：`flagtree-0.6.1+ascend3.5-cp311-cp311-linux_aarch64.whl`
- Wheel SHA256：`7d217d091bbb88104cbcfe9cbb5420827ea272250aedd78b3f57493389588df9`
- Final provider：FlagTree-owned Triton 3.5.1 / Ascend backend。

### FlagGems

- Official repo：`https://github.com/flagos-ai/FlagGems`
- Tag/commit：`v5.3.4` / `f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`
- Source tarball SHA256：`e6ebfda604a4496e80d879a2a2082dd7a11100ae19ac580d7708dace92337a4c`
- Patched wheel SHA256：`f3f48e8d2a122ae28092cb611752f05fb920fa6fa4a0e67b126b75ac91d958c7`
- Build/install entry：

```bash
COMPILER=flagtree FLAGGEMS_BACKEND=ascend-cann900 \
SETUPTOOLS_SCM_PRETEND_VERSION_FOR_FLAG_GEMS=5.3.4 \
python -m pip install --no-build-isolation --no-deps .
```

Runtime vendor选择为`GEMS_VENDOR=ascend`；它应由重建脚本以有Evidence的方式设置或验证。

### vllm-plugin-FL

- Formal repo：`https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos`
- Branch/commit/tree：`project/glm52-w8a8-v024` / `a9435a34dcd7d0a38e3a853535947371a6c62205` / `e5e073edf4b65c053e954d78d20365aab0e1f46b`
- 从保留`.git`的writable staging验证HEAD/tree/clean后安装：

```bash
python -m pip install --no-deps --no-build-isolation -e .
```

- FL不得patch；original run的Code change/PR=`N/A`。

## Third-party patches

两个patch只作用于third-party source，必须在clean unmodified failure自然复现后重放：

| Patch | Formal pointer | Known identity | Replay status |
|---|---|---|---|
| FlagTree `do_bench_npu` lazy import | Original Evidence `patches/flagtree-do-bench-npu-lazy-import.patch` | Patch SHA256 `b812a72728f008d8f6296311a44a5b444076e6764ccc5276c5554b9c5fa6f9fe` | exact target/cwd/strip level、input/output hash和canonical replay command待final verification恢复 |
| FlagGems DSA package init | Original Evidence `patches/flaggems-dsa-package-init.patch` | Patched wheel SHA256见上；patch-file hash待恢复 | exact target/cwd/strip level、input/output hash和canonical replay command待final verification恢复 |

Replay入口来自：

- Original Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z/`
- Supplemental mapping：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z-supplemental-codex2/acceptance-evidence-index.md`

Final verification须输出checksum-bound的patch replay script/command；不得靠本文猜测`patch -pN`、working directory或target path。

## Rebuild order

从container丢失状态恢复时，按以下大致顺序；精确命令/文件路径以final verification产生的脚本和lock为准：

1. Pull confirmed base tag并验证frozen RepoDigest。
2. 用现场恢复的`rebuild-container.sh`创建container，逐项核对device/mount/network/shm/security/resource配置。
3. Inventory base Python/CANN/A3 ops/NNAL/toolchain；不得复制其他container的runtime。
4. 从checksum-bound artifact安装torch/torch_npu 2.10 tuple。
5. 从exact source构建/安装vLLM 0.24 empty。
6. 安装exact FlagTree wheel；按captured replay处理lazy-import patch并验证single provider。
7. 获取exact FlagGems v5.3.4；按captured replay处理DSA package-init patch，构建/安装patched wheel。
8. 从formal FL Git branch创建writable staging，验证exact HEAD/tree/clean后editable安装；不修改FL。
9. 核对最终tuple、package/import origins、provider ownership和patch/artifact hashes。
10. 只按相应验收合同运行最小smoke；模型/E2E/性能不属于环境重建证明。

## Evidence / Control pointers

- Original task：[`tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`](tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md)
- Immutable result：[`results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md`](results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md)
- Supplemental pointer：[`results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z-evidence-supplement-codex2.md`](results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z-evidence-supplement-codex2.md)
- Final verification task：[`tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION.md`](tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION.md)
- Final verification Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION/<run-id>/`（执行后填run-id）
- Result index：[`results/INDEX.md`](results/INDEX.md)

Original run + supplemental Evidence + final verification Evidence联合用于A2最终Acceptance。本文建立rebuild baseline，不代表现有runtime已重建验证，也不改变当前`NEEDS-FOLLOWUP`。
