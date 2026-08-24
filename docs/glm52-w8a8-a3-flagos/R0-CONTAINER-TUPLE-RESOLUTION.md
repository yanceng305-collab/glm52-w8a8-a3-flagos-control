# Container R0-clean Tuple Resolution

状态：**Superseded as formal route**；compatibility/reference evidence retained

决策日期：2026-08-21

## Supersession notice

本文件记录control commit `c70aa4b83b4d270ee1d920e807296d0b283cfab2`在“vllm-ascend image/package存在本身即违规”前提下形成的neutral-base tuple研究。该前提已被后续official runtime ownership复核推翻：R0-P1/R0-F1不再是当前正式路线，neutral CANN base也不是唯一合法选择。下列OCI digest、CANN/torch-npu兼容性与Host/Container边界可继续作为reference evidence；所有“必须从未安装”“negative audit为合规验收”“primary/fallback已冻结”的表述均按本notice视为**Superseded**。

后续`92a6f767...` / `v0.20.2rc1-a3`优先候选也已被official branch migration Superseded为maintenance/reference。当前A2 artifact决策见[`OFFICIAL-V024-BASELINE-RESEARCH.md`](OFFICIAL-V024-BASELINE-RESEARCH.md)：documented `v0.24.0rc1-a3` tag无可用artifact，使用exact digest的official release-branch A3 nightly作provisional carrier且runtime tuple由preflight冻结。carrier/package存在仍不是单独PASS/FAIL条件。

## 决策边界

本项目采用容器化部署。Host与Container按以下边界治理：

```text
Host
  A3/910C hardware + 16×64GB logical devices
  Driver 25.5.0 + Firmware 7.8.0.5.216
  Docker / Ascend container runtime
  device、driver/DCMI工具与网络挂载条件
  HCCN/HCCL Host网络
  disk/model paths与NPU占用
    ↓
Container
  OS + Python + CANN Toolkit/ops/NNAL
  torch + torch-npu
  Triton-Ascend或FlagTree
  vLLM empty + vllm-plugin-FL
  FlagGems；FlagCX后置
```

Host已安装CANN 8.5.0/9.0.1不参与R0版本选择。只有未来设计显式bind-mount Host Toolkit时才重新进入兼容判断；R0-clean原则上只挂Host Driver/DCMI工具和device/network条件，不挂Host CANN Toolkit。

Host事实来自用户确认，验证级别为`user-confirmed`；本轮没有连接服务器或运行container。

## 本机candidate base静态审查

Candidate：`quay.io/ascend/cann:9.0.1-a3-ubuntu22.04-py3.12`

| 项目 | 只读OCI结果 | 状态 |
|---|---|---|
| Multi-arch index digest | `sha256:0fdf5a67b12f1173800f874abf9ad3e20d20c3327d14da53f7d0fbf62ea87aaf` | Confirmed from registry |
| ARM64 manifest digest | `sha256:004a33d8634c191399418ea81e0c6a367b7459a5713c206c4f9088294337f149` | Confirmed from registry |
| ARM64 config digest | `sha256:1572a3447f8eb74778d9956cb79cb38c39734f81b20ed58f5d9f0ba04c78f807` | Confirmed from registry |
| OS / architecture | Ubuntu 22.04 label / linux arm64 | Confirmed |
| Python | `/usr/local/python3.12.13` | Confirmed |
| CANN | `/usr/local/Ascend/cann-9.0.1`；Toolkit/ops路径 | Confirmed |
| NNAL/ATB | `/usr/local/Ascend/nnal/atb/latest/atb/cxx_abi_1` | Confirmed |
| Base history | Ubuntu layer + build tools + copiedPython/CANN；未显示torch、vLLM或vllm-ascend安装层 | Static-checked；runtime absence仍Unknown |
| Entrypoint | source container Toolkit/BiSheng/NNAL env后执行命令 | Confirmed |

历史结论：该image是可研究的neutral base candidate。**Superseded：** 它不再是R0-primary或唯一合法base；package/module/entry-point absence也不再是合规验收。OCI digest仍可用于后续对照实验。

## 历史候选：`R0-P1-CANN901-PY312`（Superseded as primary）

历史用途：曾作为Clean Provenance与official 910C Qwen canary第一候选；现不再具有执行优先级，也不声称解决GLM-5.2的vLLM>=0.23语义和mandatory capability缺口。

| 层 | 冻结值 | 依据 / 状态 |
|---|---|---|
| Base image | `quay.io/ascend/cann:9.0.1-a3-ubuntu22.04-py3.12`；执行前pin ARM64 digest`sha256:004a33...` | OCI Confirmed；本机digest待核对 |
| Container OS/arch | Ubuntu 22.04 / arm64 | OCI Confirmed |
| Python | 3.12.13 | OCI Confirmed；vLLM0.20.2与FL均声明Python3.12支持 |
| CANN Toolkit + ops | 9.0.1，来自base | OCI Confirmed |
| NNAL/ATB | base内9.0.1配套路径，CXX ABI 1 | OCI/reference Confirmed；exact package inventory待验 |
| torch | `2.10.0` | official vLLM-Ascend v0.23.0rc1 A3 tuple |
| torch-npu | `2.10.0.post2` | official vLLM-Ascend v0.23.0rc1 A3 tuple |
| Compiler | `triton-ascend==3.2.1`；不装FlagTree | official vLLM-Ascend v0.23.0rc1 tuple；FlagTree另profile |
| vLLM | official `v0.20.2@bc150f50299199599673614f80d12a196f377655`，`VLLM_TARGET_DEVICE=empty` | FL baseline/CI contract；与CANN901组合为Inferred |
| vllm-plugin-FL | `92a6f7670465922c60e88f06787b8f0923e761f3`，Python-only，`VLLM_VENDOR` unset | 正式代码baseline Confirmed |
| FlagGems | `3e6528cf04f5f964a7b0fa6628de6f0410dbfd02`，default install，不用`[ascend]`extra | FL Ascend Docker pin；preferred非mandatory |
| compressed-tensors | `0.15.0.1` | official vLLM0.20.2 common requirement |
| transformers | `5.5.3` | 保持FL/vLLM0.20.2 actual CI pin；vLLM0.20.2静态约束允许，runtime待验 |
| NumPy | `1.26.4` | FL current 910C CI assertion |
| Build prerequisites | clang-15、gcc/g++、cmake>=3.26、ninja及FL build deps | official A3/FL build source；exact versions待记录 |
| FlagCX | 不安装/不激活 | 首次bring-up后置 |
| FlagScale | 不安装/不前置 | 控制编排层后置 |
| vllm-ascend | 历史方案要求从未安装 | **Superseded；presence不再是门禁** |
| Runtime selectors | `VLLM_PLUGINS=fl`、`VLLM_FL_PLATFORM=ascend`、`VLLM_VENDOR` unset | FL source Confirmed |

历史选择理由：本机已有candidate base与较新的official A3软件栈一致；CANN901/Python312/torch-npu post2由official vLLM-Ascend v0.23.0rc1作为Ascend hardware reference；vLLM/FL保持0.20.2接口基线。该组合仍可作为**Inferred reference tuple**，但不再自动进入实验。

## 历史候选：`R0-F1-CANN900-PY311`（Superseded as fallback）

历史规则曾规定仅在R0-P1失败且归因到CANN901/Python312/torch-npu post2时启用；该自动fallback规则现已Superseded。

| 层 | 冻结值 |
|---|---|
| Base image | `quay.io/ascend/cann:9.0.0-a3-ubuntu22.04-py3.11`；ARM64 manifest`sha256:79dda763508955fd70a3a84f233aba4cb2b2e2391bd4594aaaf80dcb0c688947` |
| OS/Python/CANN | Ubuntu22.04 / Python3.11.15 / CANN9.0.0 + matching NNAL/ATB |
| torch/torch-npu | 2.10.0 / 2.10.0 |
| Compiler | triton-ascend3.2.1；无FlagTree |
| vLLM/FL/FlagGems | vLLM0.20.2 empty / FL`92a6f767...` / FlagGems`3e6528cf...` |
| transformers/NumPy | 5.5.3 / 1.26.4 |
| FlagCX/FlagScale | 不安装、不前置 |
| vllm-ascend | 历史方案要求从未安装；该门禁已Superseded |

Fallback依据是official FL 910C CI实际软件链及vLLM-Ascend v0.20.2rc1镜像构建的下层tuple；它只复刻CI-equivalent-minus-vllm-ascend，不代表Host CANN fallback。CANN8.5.0不是R0 fallback。

## 官方证据链

1. FL baseline `92a6f767...`仍声明official vLLM0.20.2和Ascend FlagGems pin`3e6528cf`：[FL Dockerfile](https://github.com/flagos-ai/vllm-plugin-FL/blob/92a6f7670465922c60e88f06787b8f0923e761f3/docker/ascend/Dockerfile)。
2. vLLM-Ascend `v0.23.0rc1@f4a08bdd...`的A3 Dockerfile使用同一CANN901/Python312 base、`SOC_VERSION=ascend910_9391`、vLLM0.23 empty和triton-ascend3.2.1：[Dockerfile.a3](https://github.com/vllm-project/vllm-ascend/blob/v0.23.0rc1/Dockerfile.a3)。
3. 同一release requirements固定torch2.10.0、torch-npu2.10.0.post2、triton-ascend3.2.1和transformers5.5.4：[requirements](https://github.com/vllm-project/vllm-ascend/blob/v0.23.0rc1/requirements.txt)。
4. TorchNPU官方安装示例把torch2.10.0/torch-npu2.10.0.post2与CANN9.0系列配套：[Ascend/pytorch README](https://github.com/Ascend/pytorch/blob/master/README.zh.md)。
5. CANN9.0.1 release notes定义Toolkit、ops、NNAL组合与Driver配套章节：[CANN9.0.1 release notes](https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/900/releasenote/9.0.1release-notes.md)。
6. 华为容器文档明确Host安装Driver/Firmware并把device和Driver目录挂入container，而CANN软件位于container：[Host mount guidance](https://www.hiascend.com/document/detail/zh/canncommercial/700/envdeployment/instg/instg_0099.html)。
7. Fallback tuple来自[vLLM-Ascend v0.20.2rc1 requirements](https://github.com/vllm-project/vllm-ascend/blob/v0.20.2rc1/requirements.txt)和[Dockerfile.a3](https://github.com/vllm-project/vllm-ascend/blob/v0.20.2rc1/Dockerfile.a3)。

修正：vLLM-Ascend仍可作为hardware implementation reference；其official A3 image也可作为FlagOS环境carrier。是否有`vllm_ascend`实际runtime参与必须由trace回答，不能由package inventory推断。

## Unknown与实验门禁

- 本机candidate image RepoDigest是否等于本次registry ARM64 digest；tag可能移动；
- official carrier exact digest、package/native内容与本机tag一致性未知；
- Driver25.5.0 + Firmware7.8.0.5.216与CANN901 base的exact公开配套表未被本轮独立解析，需最小device/import诊断确认；
- FL`92a6f767` + vLLM0.20.2在CANN901/Python312/torch-npu post2组合上的import、worker初始化和Qwen canary未验证；
- transformers5.5.3与FL tokenizer/config patches在CANN901 primary中的真实闭环未验证；
- FlagGems`3e6528cf`在CANN901/triton-ascend3.2.1下的preferred operator coverage与fallback路径未验证；
- Host device/driver/DCMI挂载是否足够覆盖16个logical devices、HCCN/HCCL和container内`npu-smi`未验证；
- neutral candidate是否仍有对照实验价值，只能由official carrier trace的具体故障决定；不得按本文件自动切换。

## 本轮非执行声明

本轮仅进行registry/GitHub/official文档静态研究并更新控制面；没有连接服务器、pull/run/create container、安装package或生成DeepSeek执行任务。
