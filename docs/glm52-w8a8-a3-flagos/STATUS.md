# 项目状态

更新时间：2026-08-21
总体状态：Research Freeze accepted；A2 scope reprioritized；execution not started by design

## 当前快照

| 工作项 | 状态 | 证据边界 |
|---|---|---|
| 官方 FlagOS 环境/CI 调查 | Complete for current static source | current FL `92a6f767`、vLLM 0.20.2、公开 CI/OCI；早期CI证据固定`38e7dbc` |
| Runtime ownership | Static model complete；full dynamic audit deferred | current FL `92a6f767`静态追踪 + 910C Qwen CI；完整import/call/native/provider审计位于Eager Correctness后 |
| 910C maturity | Complete for public evidence | 仅 Qwen3.6 TP2；不覆盖 GLM/W8A8/MTP/EP/多机 |
| GLM-5.2-W8A8 compatibility | Complete static assessment | 当前多个 Missing；没有目标 E2E |
| A3 capacity/topology | User-confirmed boundary | 16×64GB logical devices / 1024GB aggregate；runtime reservation与full-model余量Unknown |
| Formal A3 code repository | Established / verified | Standalone `vllm-plugin-FL-a3-flagos`；main/tree与official冻结基线一致 |
| Legacy preservation | Verified unchanged | 12 branches、5 tags、PR #1和settings前后快照一致 |
| Official carrier FL-only environment | A2 Prepared / Not Ready | 只在一次性实验container卸载vllm-ascend并做最小smoke；尚无服务器授权 |
| GLM migration code | Not started | 按用户要求 |
| Performance optimization | Not started | 必须在 correctness/baseline 后 |
| Host facts for container boundary | User-confirmed | 16×64GB topology、Driver25.5.0、Firmware7.8.0.5.216及container runtime约束；未由Codex现场验证 |
| Neutral R0 tuple research (`c70aa4b`) | **Superseded as formal route** | R0-P1/R0-F1仅保留compatibility reference evidence |
| Post-Eager Runtime Provenance Audit | Deferred / On-demand / Not Ready | 原A2完整trace设计已保留；Eager Correctness后按触发条件做A/B审计，不阻塞Baseline Benchmark |

## 已确认的高影响事实

- 官方FL Ascend Dockerfile使用`quay.io/ascend/vllm-ascend:v0.20.2rc1-a3`作环境carrier；CI没有卸载其package。
- `VLLM_PLUGINS=fl`使实际platform为FL；静态链路继续到`WorkerFL`、`ModelRunnerFL`与FlagOS Dispatch。
- official `ascend.yaml`按operator选择FlagGems、`vendor.ascend`或Reference；`vendor.ascend`位于`vllm_fl` ownership并直接调用torch_npu/CANN，不是vllm-ascend backend wrapper。
- vllm-ascend package存在本身不再判违规；目标进程中是否有任何`vllm_ascend`实际import/call仍Unknown。
- A2中的container内卸载只用于减少FL-only bring-up变量，不构成对official coexistence路线的合规否定。
- FL editable安装不得直接写readonly正式repo；如确有需要，必须在container内复制含`.git`的writable staging，安装前验证exact HEAD/tree/clean状态，生成artifact只留副本。
- 卸载后的distribution、`find_spec`和entry-point negative check必须由新的Python process执行。
- 官方当前 910C 成功证据是 Qwen3.6-27B/35B-A3B TP2；GLM、W8A8、MTP、EP、多机、FlagCX均未覆盖。
- FL current main 使用 vLLM 0.20.2；GLM-5.2 官方要求 0.23.0+ 的 IndexShare/MTP reuse。
- 当前 FL Ascend 上 usable MLA、DSA/SFA、wired NPU Indexer、W8A8 Linear、AscendV1 reader 为 Missing。
- 默认通信 backend 是 HCCL；FlagCX optional。
- 当前 FL Ascend 不构建自身 native extension；`VLLM_VENDOR` 必须 unset，设置 `ascend` 会失败。

## 当前阻塞/待决策

1. official A3 carrier是否已存在本机及其exact RepoDigest/image ID尚需执行时确认；缺失或digest不确定时不得pull。
2. 第一台A3当前有其他OC2 benchmark任务；A2执行时是否存在明确空闲的最小logical device范围必须重新检查。
3. FL-only container中的minimal negative check、`PlatformFL/WorkerFL/ModelRunnerFL/Dispatch`与synthetic NPU operator尚未实验验证。
4. coexistence模式下关键operator、active compiler provider及任何`vllm_ascend`动态参与有意后置为Post-Eager Audit，当前为Unknown但不阻塞A2/canary。
5. GLM vLLM路线：0.23最小升级、0.24 dev/未合入Ascend PR，或0.20.2 backport。
6. 客户checkpoint是AscendV1还是compressed-tensors；manifest、SHA、tensor/scale layout未提供。

当前“FlagOS原生”工作边界按实际模型执行ownership判定，不按carrier image/package存在性判定。若trace发现`vllm_ascend`实际参与，只对具体调用进入客户边界/替换判断；只有客户以后明确扩大到official FL历史adapted来源时才另行重审源码合规。

## GitHub 状态

- `yanceng305-collab/vllm-plugin-FL` 已是官方 fork network 的 fork；direct parent 是 `xiemingda-1002/vllm-plugin-FL`，network source 是 `flagos-ai/vllm-plugin-FL`。
- PR #1：open Draft；base `ascend-model-migration`，head `audit/glm52-w8a8-stage0-gap`。
- 正式代码仓库：[yanceng305-collab/vllm-plugin-FL-a3-flagos](https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos)，类型为standalone而非fork。
- Official frozen baseline：commit `92a6f7670465922c60e88f06787b8f0923e761f3`，tree `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`；新仓库main/tree完全相同。
- 新仓库只有`main`且无tag；selected legacy-only commits不可达。
- Legacy未执行rename、transfer、detach、fork sync、push或settings修改；branches/tags/PR #1哈希前后相同。
- 新仓库`main`禁止直接开发；official同步只按`CODE-REPOSITORY-BASELINE.md`的control-approved fast-forward policy执行。

## 下一门禁

下一建议任务是[`tasks/STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](tasks/STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)：用户另行批准后，由DeepSeek在第一台A3上使用本机已有exact official carrier创建一次性实验container，完成受控卸载、minimal negative check、FL class/dispatch闭环和一个synthetic NPU operator smoke。当前Prepared / Not Ready；本轮未操作服务器、未创建container、未生成DeepSeek执行提示词。

原完整trace设计已后置到[`tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md`](tasks/POST-EAGER-RUNTIME-PROVENANCE-AUDIT.md)，定义为Eager Correctness后的Deferred / On-demand审计支线，不是Baseline Benchmark硬门禁。

A2当前没有未解决的control-contract blocker；执行仍Not Ready，因为尚未获得服务器/DeepSeek授权，并且本机exact carrier identity与OC2占用下的空闲logical device必须在执行preflight确认。
