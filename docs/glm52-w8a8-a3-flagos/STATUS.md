# 项目状态

更新时间：2026-08-24
总体状态：上一轮PY311 smoke为**feasibility proven / formal clean-room validation required**；当前Ready任务为Ascend官方CANN 9.0.0 A3 clean-room复现

## 当前快照

| 工作项 | 状态 | 证据边界 |
|---|---|---|
| Official branch migration | PASS | `v0.2.1@92a6f767`维护线；`main@a9435a34`/tree`e5e073ed`冻结为0.24 baseline |
| Runtime ownership | Static architecture retained；new-main dynamic unverified | current main仍注册PlatformFL/WorkerFL/ModelRunnerFL/Dispatch；server未执行 |
| 910C maturity | Complete for public evidence | v0.2.1-era仅Qwen3.6 TP2；new v0.24 stack无FL A3 E2E |
| GLM-5.2-W8A8 compatibility | Complete static assessment | 当前多个 Missing；没有目标 E2E |
| A3 capacity/topology | User-confirmed boundary | 16×64GB logical devices / 1024GB aggregate；runtime reservation与full-model余量Unknown |
| Formal A3 code repository | Migration PASS | existing main不变；v0.2.1 anchor、v0.24 anchor、project branch exact |
| Repository/Evidence governance | **Implemented** | `REPOSITORY-AND-EVIDENCE-RULES.md` + `results/INDEX.md` |
| A3 Host directory initialization | **ACCEPTED with deviation** | Result `A3-HOST-DIR-INIT/20260824T030000Z`；两个长期repo identity PASS；legacy复制偏差已记录，原目录未改且现有副本不处理 |
| Legacy preservation | Verified unchanged | 12 branches、5 tags、PR #1和settings前后快照一致 |
| Old v0.20.2 A2/prompt | **Superseded / Paused / not executed** | `118c314` prompt不得下发服务器 |
| A2 environment gate | **STOP at FlagTree (Phase A) / execution paused** | Immutable result：[`results/A2-v024/20260824T025250Z.md`](results/A2-v024/20260824T025250Z.md)；Codex technical acceptance PENDING |
| A2 shared-NPU tiny smoke | **Not reached (Phase A STOP)** | 未进入Phase B；NPU 12+13状态未采集 |
| PY311 copied-runtime smoke | **Feasibility proven / not formally accepted** | Qwen image复制Python/FlagTree/vLLM并使用3处临时patch；immutable PASS保留，但不能作为正式环境acceptance |
| CANN 9.0.0 A3 clean-room | **Ready after user dispatch** | AscendHub official pull ref first；fallback official Dockerfile`aec636189a23...`；禁止复用上一轮runtime |
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
- Official文档定义的A3 release tag仍为`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`，但当前未建立可用artifact；该tag不得用于本次A2。
- A2唯一provisional carrier为`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`，身份仅为official `releases/v0.24.0rc` A3 nightly，不是rc1 release image。
- 不预设该carrier内部CANN、torch-npu、vLLM、triton-ascend或其他runtime tuple；实际版本由container preflight冻结。
- A2 tiny smoke允许共享NPU 12+13；现有任务继续运行，状态恶化或可能干扰时立即STOP。
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
- Old A2 FlagGems`no-deps`/禁止bootstrap限制不适用于D-075 clean-room；允许clean container内安装对齐依赖、build wheel和保存patch provenance。
- 服务器缺formal FL source时可direct clone唯一repo/project branch；GitHub不可达时允许expected SHA256校验过的Git bundle relay。最终必须验证branch=`project/glm52-w8a8-v024`、HEAD=`a9435a34...`、tree=`e5e073ed...`、worktree clean。
- Current-main FL Ascend上MLA、DSA/SFA、wired Indexer与MLA cache closure仍Missing/Unwired；W8A8 Linear必须基于vLLM0.24重新gap confirmation，当前无910C PASS。
- 默认通信 backend 是 HCCL；FlagCX optional。
- 当前 FL Ascend 不构建自身 native extension；`VLLM_VENDOR` 必须 unset，设置 `ascend` 会失败。

## 执行时必须验证 / 后续待决策

1. 从AscendHub官方pull信息取得CANN9.0.0 A3 py311 devel完整image ref并冻结digest；不得猜registry。
2. 无法取得official image时，只允许从`Ascend/cann-container-image@aec636189a23...`目标Dockerfileclean build同一9.0.0基线并冻结digest。
3. 全栈必须独立安装；禁止Qwen image/runtime复制、跨Python复制vLLM和复用上一轮container/work。
4. 上一轮3处patch必须先在clean unmodified source中重现/未重现；FL patch需要task branch/commit/PR。
5. FlagTree必须single coherent provider；环境闭合后仅用NPU 12+13执行tiny tensor/kernel/Dispatch。
6. CANN9.0.0明确无法合理适配时STOP，不得自行切换到9.0.1。

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

上一轮PY311 PASS只保留feasibility价值，不解锁正式A2。下一项Ready task：

- Task：[`tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`](tasks/STAGE-A2-V024-CLEANROOM-CANN900-PY311.md)
- Prompt：[`tasks/DEEPSEEK-A2-V024-CLEANROOM-CANN900-PY311-PROMPT.md`](tasks/DEEPSEEK-A2-V024-CLEANROOM-CANN900-PY311-PROMPT.md)

Clean-room固定CANN 9.0.0 A3 Python3.11 devel基线，不猜registry地址、不复制Qwen image/runtime、不复用上一轮container/work、不预应用3处patch。完成独立安装与tiny smoke后再决定是否正式接受为A2基础环境。
