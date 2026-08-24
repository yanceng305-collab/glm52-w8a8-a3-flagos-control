# 项目状态

更新时间：2026-08-21
总体状态：v0.24 repository migration PASS；A3-CP-A2-v024 Ready / not executed

## 当前快照

| 工作项 | 状态 | 证据边界 |
|---|---|---|
| Official branch migration | PASS | `v0.2.1@92a6f767`维护线；`main@a9435a34`/tree`e5e073ed`冻结为0.24 baseline |
| Runtime ownership | Static architecture retained；new-main dynamic unverified | current main仍注册PlatformFL/WorkerFL/ModelRunnerFL/Dispatch；server未执行 |
| 910C maturity | Complete for public evidence | v0.2.1-era仅Qwen3.6 TP2；new v0.24 stack无FL A3 E2E |
| GLM-5.2-W8A8 compatibility | Complete static assessment | 当前多个 Missing；没有目标 E2E |
| A3 capacity/topology | User-confirmed boundary | 16×64GB logical devices / 1024GB aggregate；runtime reservation与full-model余量Unknown |
| Formal A3 code repository | Migration PASS | existing main不变；v0.2.1 anchor、v0.24 anchor、project branch exact |
| Legacy preservation | Verified unchanged | 12 branches、5 tags、PR #1和settings前后快照一致 |
| Old v0.20.2 A2/prompt | **Superseded / Paused / not executed** | `118c314` prompt不得下发服务器 |
| New v0.24 A2 | **Ready / not executed** | exact carrier/network/compiler replacement/valid pair纳入task PASS/STOP；prompt已生成 |
| GLM migration code | Not started | 按用户要求 |
| Performance optimization | Not started | 必须在 correctness/baseline 后 |
| Host facts for container boundary | User-confirmed | 16×64GB topology、Driver25.5.0、Firmware7.8.0.5.216及container runtime约束；未由Codex现场验证 |
| Neutral R0 tuple research (`c70aa4b`) | **Superseded as formal route** | R0-P1/R0-F1仅保留reference evidence |
| Post-Eager Runtime Provenance Audit | Deferred / On-demand / Not Ready | 原A2完整trace设计已保留；Eager Correctness后按触发条件做A/B审计，不阻塞Baseline Benchmark |

## 已确认的高影响事实

- Historical v0.2.1 evidence：当时official FL Ascend路线使用`v0.20.2rc1-a3` carrier且未卸载package；该证据仍支撑runtime-ownership原则，但不再定义primary tuple。
- 上述`92a6f767`路线现在属于official `v0.2.1`/vLLM0.20.2 maintenance line，不再是GLM primary。
- official current `main`本轮冻结为`a9435a34...`/tree`e5e073ed...`，pyproject/README固定vLLM0.24；`main`未来可能前进。
- official `v0.2.1`与new `main`已diverged，不能fast-forward或merge成伪升级。
- `bb439d...`在NVIDIA A800上完成GLM-5.2-Slim TP16双机init和weight loading后因MLA cache op缺失而PARTIAL；不是Ascend/W8A8 E2E。
- new primary carrier candidate为`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`；source tuple为CANN9.0.1/Py3.12/vLLM0.24/torch2.10/torch-npu post2/triton-ascend3.2.1。
- A3 official v0.24 docs要求至少两个logical devices；valid/same-card pair映射需Host证据，不能猜ID。
- current README选择FlagGems v5.3.4与FlagTree`0.6.1rc1+ascend3.5`；FlagTree与triton-ascend共享完整`triton`namespace，是替代provider而非clean coexistence。
- current main Ascend Dockerfile仍为0.19/CANN8.5/old compiler tuple，标记Upstream Conflict / stale candidate。
- `VLLM_PLUGINS=fl`使实际platform为FL；静态链路继续到`WorkerFL`、`ModelRunnerFL`与FlagOS Dispatch。
- official `ascend.yaml`按operator选择FlagGems、`vendor.ascend`或Reference；`vendor.ascend`位于`vllm_fl` ownership并直接调用torch_npu/CANN，不是vllm-ascend backend wrapper。
- vllm-ascend package存在本身不再判违规；目标进程中是否有任何`vllm_ascend`实际import/call仍Unknown。
- A2中的container内卸载只用于减少FL-only bring-up变量，不构成对official coexistence路线的合规否定。
- FL editable安装不得直接写readonly正式repo；如确有需要，必须在container内复制含`.git`的writable staging，安装前验证exact HEAD/tree/clean状态，生成artifact只留副本。
- 卸载后的distribution、`find_spec`和entry-point negative check必须由新的Python process执行。
- v0.2.1-era 910C成功证据是Qwen3.6-27B/35B-A3B TP2；new main/v0.24 carrier/FlagTree profile尚无对应FL A3 E2E，canary需重冻。
- FL new main使用vLLM0.24并已包含GLM model/Indexer结构；v0.2.1维护线仍是0.20.2。Target Ascend/W8A8 closure仍未验证。
- v0.2.1-era FL compressed-tensors validator与packed W8A8 glue不在observed new main；0.24 W8A8 loading owner必须从upstream/runtime与真实artifact重审。
- A2 FlagGems安装禁止setup/bootstrap profile，只能使用current container Python对exact v5.3.4 source执行no-build-isolation/no-deps安装。
- 服务器缺formal FL source时允许唯一repo/唯一project branch clone到Evidence `source/`并验证exact HEAD/tree/clean。
- Current-main FL Ascend上MLA、DSA/SFA、wired Indexer与MLA cache closure仍Missing/Unwired；W8A8 Linear必须基于vLLM0.24重新gap confirmation，当前无910C PASS。
- 默认通信 backend 是 HCCL；FlagCX optional。
- 当前 FL Ascend 不构建自身 native extension；`VLLM_VENDOR` 必须 unset，设置 `ascend` 会失败。

## 执行时必须验证 / 后续待决策

1. A2 task内需冻结carrier RepoDigest/image ID；本机缺失时只允许pull exact v0.24 tag。
2. A2 task内需完成FlagTree replacement并证明single coherent provider；失败STOP。
3. A2 preflight需冻结valid two-logical-device pair与OC2占用；没有完整free pair即STOP。
4. FlagGems v5.3.4 + FlagTree + FL project branch的910C synthetic smoke尚待执行。
5. target GLM-5.2-W8A8 checkpoint manifest、format、SHA、tensor/scale layout尚未提供，但不阻塞A2。
6. current-main Ascend MLA、DSA/SFA、Indexer、W8A8 Linear/MoE及`concat_and_cache_mla*` closure归后续GLM gate，不阻塞A2。

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
- Existing `main`禁止直接开发或同步覆盖；new-main migration只按`CODE-REPOSITORY-MIGRATION-V024.md`的新增branch方案，在用户另行批准后执行。

## 下一门禁

当前Ready服务器任务：[`tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)。执行提示词：[`tasks/DEEPSEEK-A3-CP-A2-V024-EXECUTION-PROMPT.md`](tasks/DEEPSEEK-A3-CP-A2-V024-EXECUTION-PROMPT.md)。

Old v0.20 task/prompt继续Paused，不得下发。

原完整trace设计已后置到[`tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`](tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md)，定义为Eager Correctness后的Deferred / On-demand审计支线，不是Baseline Benchmark硬门禁。

本轮未操作服务器。代码repo branch migration与control/prompt已完成；下一步直接执行A2-v024。
