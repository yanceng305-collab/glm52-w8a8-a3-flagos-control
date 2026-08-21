# 项目状态

更新时间：2026-08-21
总体状态：Research complete for frozen sources；execution not started by design

## 当前快照

| 工作项 | 状态 | 证据边界 |
|---|---|---|
| 官方 FlagOS 环境/CI 调查 | Complete | 固定 FL `38e7dbc`、vLLM 0.20.2、公开 CI/OCI |
| Runtime ownership | Complete | 固定源码静态追踪 + 910C Qwen CI |
| 910C maturity | Complete for public evidence | 仅 Qwen3.6 TP2；不覆盖 GLM/W8A8/MTP/EP/多机 |
| GLM-5.2-W8A8 compatibility | Complete static assessment | 当前多个 Missing；没有目标 E2E |
| A3 capacity/topology | User-confirmed boundary | 16×64GB logical devices / 1024GB aggregate；runtime reservation与full-model余量Unknown |
| Formal A3 code repository | Established / verified | Standalone `vllm-plugin-FL-a3-flagos`；main/tree与official冻结基线一致 |
| Legacy preservation | Verified unchanged | 12 branches、5 tags、PR #1和settings前后快照一致 |
| Customer-compliant environment | Planned / Inferred | `R0-clean` 尚未在 910C 执行 |
| GLM migration code | Not started | 按用户要求 |
| Performance optimization | Not started | 必须在 correctness/baseline 后 |
| Host facts for container boundary | User-confirmed | 16×64GB topology、Driver25.5.0、Firmware7.8.0.5.216及container runtime约束；未由Codex现场验证 |
| Container R0 tuple resolution | Static complete | Primary CANN901/Python312；conditional fallback CANN900/Python311 |
| Clean environment build | Not Ready | 等待local image digest/negative audit实验与用户另行批准 |

## 已确认的高影响事实

- 官方 A3 CI image 是完整 vllm-ascend runtime；CI 没有卸载/覆盖其 package。
- CI 中 `vllm-ascend` installed/discoverable，但 `VLLM_PLUGINS=fl` 使实际 platform 为 FL。
- 官方当前 910C 成功证据是 Qwen3.6-27B/35B-A3B TP2；GLM、W8A8、MTP、EP、多机、FlagCX均未覆盖。
- FL current main 使用 vLLM 0.20.2；GLM-5.2 官方要求 0.23.0+ 的 IndexShare/MTP reuse。
- 当前 FL Ascend 上 usable MLA、DSA/SFA、wired NPU Indexer、W8A8 Linear、AscendV1 reader 为 Missing。
- 默认通信 backend 是 HCCL；FlagCX optional。
- 当前 FL Ascend 不构建自身 native extension；`VLLM_VENDOR` 必须 unset，设置 `ascend` 会失败。

## 当前阻塞/待决策

1. 本机candidate image RepoDigest与remote ARM64 digest是否一致，以及container内是否完全无vllm-ascend。
2. Driver25.5.0/Firmware7.8.0.5.216与CANN901 primary的最小device/import兼容尚未实验验证。
3. FL`92a6f767`/vLLM0.20.2与CANN901/Python312/torch-npu post2组合未通过clean import/worker/Qwen canary。
4. GLM vLLM路线：0.23最小升级、0.24 dev/未合入Ascend PR，或0.20.2 backport。
5. 客户checkpoint是AscendV1还是compressed-tensors；manifest、SHA、tensor/scale layout未提供。

当前“FlagOS原生”工作边界按runtime/package/environment independence执行；只有客户以后明确扩大到official FL历史adapted来源时才重审。

## GitHub 状态

- `yanceng305-collab/vllm-plugin-FL` 已是官方 fork network 的 fork；direct parent 是 `xiemingda-1002/vllm-plugin-FL`，network source 是 `flagos-ai/vllm-plugin-FL`。
- PR #1：open Draft；base `ascend-model-migration`，head `audit/glm52-w8a8-stage0-gap`。
- 正式代码仓库：[yanceng305-collab/vllm-plugin-FL-a3-flagos](https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos)，类型为standalone而非fork。
- Official frozen baseline：commit `92a6f7670465922c60e88f06787b8f0923e761f3`，tree `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7`；新仓库main/tree完全相同。
- 新仓库只有`main`且无tag；selected legacy-only commits不可达。
- Legacy未执行rename、transfer、detach、fork sync、push或settings修改；branches/tags/PR #1哈希前后相同。
- 新仓库`main`禁止直接开发；official同步只按`CODE-REPOSITORY-BASELINE.md`的control-approved fast-forward policy执行。

## 下一门禁

R0 Container tuple静态决策已完成。下一步只能在用户另行批准后设计最小clean image/digest/negative-audit实验；当前不操作服务器、不创建container、不生成DeepSeek任务，也不开始GLM适配或性能工作。
