# 项目状态

更新时间：2026-08-25
总体状态：`A2-V024-CLEANROOM-CANN900-PY311`已由服务器执行代理报告**PASS**并同步Control；**Codex1 Acceptance NEEDS-FOLLOWUP**（仅补现有run Evidence索引/校验），下一正式技术阶段尚未解锁

## 当前快照

| 工作项 | 状态 | 证据边界 |
|---|---|---|
| Official branch migration | PASS | Code repo `main@92a6f767`是v0.2.1维护线；official new-main freeze `a9435a34`/tree`e5e073ed`已成为0.24 baseline与project branch |
| Runtime ownership | Static chain confirmed；tiny Dispatch smoke PASS reported / Acceptance NEEDS-FOLLOWUP | frozen v0.24 source与官方source一致；动态backend/device/no-fallback仍须从原始Evidence独立校验 |
| 910C maturity | Complete for public evidence | v0.2.1-era仅Qwen3.6 TP2；new v0.24 stack无FL A3 E2E |
| GLM-5.2-W8A8 compatibility | Complete static assessment | 当前多个 Missing；没有目标 E2E |
| A3 capacity/topology | User-confirmed boundary | 16×64GB logical devices / 1024GB aggregate；runtime reservation与full-model余量Unknown |
| Formal A3 code repository | Migration PASS | existing main不变；v0.2.1 anchor、v0.24 anchor、project branch exact |
| Repository/Evidence governance | **Implemented** | `REPOSITORY-AND-EVIDENCE-RULES.md` + `results/INDEX.md` |
| A3 Host directory initialization | **ACCEPTED with deviation** | Result `A3-HOST-DIR-INIT/20260824T030000Z`；两个长期repo identity PASS；legacy复制偏差已记录，原目录未改且现有副本不处理 |
| Legacy preservation | Verified unchanged | 12 branches、5 tags、PR #1和settings前后快照一致 |
| Old v0.20.2 A2/prompt | **Superseded / Paused / not executed** | `118c314` prompt不得下发服务器 |
| Historical v0.24 Python3.12 A2 | **STOP at FlagTree (Phase A)** | Immutable result：[`results/A2-v024/20260824T025250Z.md`](results/A2-v024/20260824T025250Z.md)；由后续Python3.11路线取代，不得当作当前task |
| Clean-room shared-NPU tiny smoke | **PASS reported / Acceptance NEEDS-FOLLOWUP** | 结果叙述一致；原始device/backend/exit-status/no-fallback日志尚未形成可审计索引 |
| PY311 copied-runtime smoke | **Feasibility proven / not formally accepted** | Qwen image复制Python/FlagTree/vLLM并使用3处临时patch；immutable PASS保留，但不能作为正式环境acceptance |
| CANN 9.0.0 A3 clean-room | **Execution PASS / Control SYNCED / Codex1 Acceptance NEEDS-FOLLOWUP** | 无实质性反证；Evidence manifest/checksum与关键raw logs尚未独立闭合 |
| GLM migration code | Not started | 按用户要求 |
| Performance optimization | Not started | 必须在 correctness/baseline 后 |
| Host facts for container boundary | User-confirmed | 16×64GB topology、Driver25.5.0、Firmware7.8.0.5.216及container runtime约束；未由Codex现场验证 |
| Neutral R0 tuple research (`c70aa4b`) | **Superseded as formal route** | R0-P1/R0-F1仅保留reference evidence |
| Post-Eager Runtime Provenance Audit | Deferred / On-demand / Not Ready | 原A2完整trace设计已保留；Eager Correctness后按触发条件做A/B审计，不阻塞Baseline Benchmark |

## 已确认的高影响事实

- Historical v0.2.1 evidence：当时official FL Ascend路线使用`v0.20.2rc1-a3` carrier且未卸载package；该证据仍支撑runtime-ownership原则，但不再定义primary tuple。
- 上述`92a6f767`路线现在属于official `v0.2.1`/vLLM0.20.2 maintenance line，不再是GLM primary。
- 2026-08-21 official new-main snapshot冻结为`a9435a34...`/tree`e5e073ed...`，pyproject/README固定vLLM0.24；它是项目baseline，不等于未来持续前进的upstream HEAD。
- official `v0.2.1`与new `main`已diverged，不能fast-forward或merge成伪升级。
- `bb439d...`在NVIDIA A800上完成GLM-5.2-Slim TP16双机init和weight loading后因MLA cache op缺失而PARTIAL；不是Ascend/W8A8 E2E。
- Historical v0.24 carrier研究中的`v0.24.0rc1-a3` tag与provisional nightly digest只保留reference，不是当前clean-room run的base。
- 最新clean-room run报告使用official CANN9.0.0 A3 py311 exact digest，并实测冻结Python/CANN/torch/torch_npu/vLLM/FlagTree/FlagGems/FL tuple；该报告仍待Codex1独立审查。
- 最新run报告使用FlagGems v5.3.4与FlagTree`0.6.1+ascend3.5`，最终`triton`namespace由FlagTree拥有；不得把这条执行报告先行写成formal acceptance。
- current main Ascend Dockerfile仍为0.19/CANN8.5/old compiler tuple，标记Upstream Conflict / stale candidate。
- `VLLM_PLUGINS=fl`使实际platform为FL；静态链路继续到`WorkerFL`、`ModelRunnerFL`与FlagOS Dispatch。
- official `ascend.yaml`按operator选择FlagGems、`vendor.ascend`或Reference；`vendor.ascend`位于`vllm_fl` ownership并直接调用torch_npu/CANN，不是vllm-ascend backend wrapper。
- vllm-ascend package存在本身不再判违规；目标进程中是否有任何`vllm_ascend`实际import/call仍Unknown。
- A2中的container内卸载只用于减少FL-only bring-up变量，不构成对official coexistence路线的合规否定。
- FL editable安装不得直接写readonly正式repo；如确有需要，必须在container内复制含`.git`的writable staging，安装前验证exact HEAD/tree/clean状态，生成artifact只留副本。
- 卸载后的distribution、`find_spec`和entry-point negative check必须由新的Python process执行。
- v0.2.1-era 910C成功证据是Qwen3.6-27B/35B-A3B TP2；frozen v0.24 stack现在只有待Acceptance的tiny runtime/operator/Dispatch smoke，仍无对应FL A3模型E2E，canary需重冻。
- FL new main使用vLLM0.24并已包含GLM model/Indexer结构；v0.2.1维护线仍是0.20.2。Target Ascend/W8A8 closure仍未验证。
- v0.2.1-era FL compressed-tensors validator与packed W8A8 glue不在observed new main；0.24 W8A8 loading owner必须从upstream/runtime与真实artifact重审。
- Old A2 FlagGems`no-deps`/禁止bootstrap限制不适用于D-075 clean-room；允许clean container内安装对齐依赖、build wheel和保存patch provenance。
- 服务器缺formal FL source时可direct clone唯一repo/project branch；GitHub不可达时允许expected SHA256校验过的Git bundle relay。最终必须验证branch=`project/glm52-w8a8-v024`、HEAD=`a9435a34...`、tree=`e5e073ed...`、worktree clean。
- Current-main FL Ascend上MLA、DSA/SFA、wired Indexer与MLA cache closure仍Missing/Unwired；W8A8 Linear必须基于vLLM0.24重新gap confirmation，当前无910C PASS。
- 默认通信 backend 是 HCCL；FlagCX optional。
- 当前 FL Ascend 不构建自身 native extension；`VLLM_VENDOR` 必须 unset，设置 `ascend` 会失败。

## 最新执行与Codex1审查

- Task / Run：`A2-V024-CLEANROOM-CANN900-PY311` / `20260824T080753Z`。
- Immutable result：[`results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md`](results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md)；Server Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z`。
- Reported tuple：Python 3.11.15、CANN 9.0.0、torch 2.10.0+cpu、torch_npu 2.10.0.post2、vLLM 0.24.0、FlagTree 0.6.1+ascend3.5 / Triton 3.5.1、FlagGems 5.3.4、FL `a9435a34...`。
- Reported validation：tiny torch_npu、FlagTree kernel、FlagGems op、FL/Dispatch op PASS；FL未修改，Code PR=`N/A`。
- Reported issue handling：Triton circular import与FlagGems DSA package-init问题复现并保存third-party patch；FL `patch_mamba_config` / `cbor2`问题未复现。
- Code/source复核：formal project branch仍为`a9435a34...`/tree`e5e073ed...`，无本task远端branch或PR；official vLLM/FlagTree/FlagGems身份和两个patch target均与result一致。
- Codex1 Acceptance：**NEEDS-FOLLOWUP**。Immutable result内部一致且未发现运行失败、错误image、混合provider或FL修改反证；但当前无法只读访问Server Evidence，Control也未索引manifest/checksum，故不能从摘要替代raw Evidence完成正式验收。

当前“FlagOS原生”工作边界按实际模型执行ownership判定，不按carrier image/package存在性判定。若trace发现`vllm_ascend`实际参与，只对具体调用进入客户边界/替换判断；只有客户以后明确扩大到official FL历史adapted来源时才另行重审源码合规。

## GitHub 状态

- `yanceng305-collab/vllm-plugin-FL` 已是官方 fork network 的 fork；direct parent 是 `xiemingda-1002/vllm-plugin-FL`，network source 是 `flagos-ai/vllm-plugin-FL`。
- PR #1：open Draft；base `ascend-model-migration`，head `audit/glm52-w8a8-stage0-gap`。
- 正式代码仓库：[yanceng305-collab/vllm-plugin-FL-a3-flagos](https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos)，类型为standalone而非fork。
- Official frozen baseline：commit `92a6f7670465922c60e88f06787b8f0923e761f3`，tree `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`；新仓库main/tree完全相同。
- 该baseline现在正式标记为official `v0.2.1` maintenance/reference；new main research freeze为`a9435a34...`/tree`e5e073ed...`。
- `baseline/official-v0.2.1-vllm0.20.2` -> `92a6f767...`。
- `baseline/official-main-vllm0.24-20260821-a9435a3` -> `a9435a34...`。
- `project/glm52-w8a8-v024` -> `a9435a34...`。
- Default branch仍为existing `main`；本轮未配置protection/PR规则。
- Legacy未执行rename、transfer、detach、fork sync、push或settings修改；branches/tags/PR #1哈希前后相同。
- Existing `main`禁止直接开发或同步覆盖；v0.24 migration已完成，后续代码task从`project/glm52-w8a8-v024`派生并按合同提交PR。

## 下一门禁

当前唯一门禁是对**现有run**完成最小只读Evidence补充与独立校验；不需要重跑、重建或重新安装：

1. 导出现有Evidence的相对文件清单、manifest/checksum文件名与SHA256，并保存成功的checksum验证输出。
2. 建立“验收项 → 既有文件/行号”映射，覆盖image/container clean/no-copy与replay artifact、最终package/import/provider、两个patch的原始/patch/产物hash与重放方法、FL HEAD/tree/clean/import realpath、三个历史问题以及四个tiny NPU smoke的device/backend/exit status/no-fallback。
3. 补全三指针：Code repo + frozen branch/commit/tree + PR=`N/A`；immutable result + result commit `56b580e...` + sync marker `31d20df...`；Evidence path + manifest/checksum。

补充只能从`20260824T080753Z`已保留的artifact/log提取或计算hash；不得修改closed run或immutable result。若原Evidence目录不可变，使用单独的immutable supplemental Evidence并明确回链。

在补证完成、Codex1独立校验并更新Acceptance前：

- 不创建或下发下一技术task；
- 不开始GLM-5.2-W8A8模型适配、模型运行或性能实验；
- 不把历史Ready task/prompt重新下发；
- 下一正式技术阶段保持未解锁。
