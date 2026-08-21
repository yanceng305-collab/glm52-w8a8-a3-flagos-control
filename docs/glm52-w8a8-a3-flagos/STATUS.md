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
| A3 capacity | Complete static assessment | 单机 1024 GB aggregate；现场 topology 与 runtime余量 Unknown |
| Repository feasibility | Complete read-only audit | 零 GitHub mutation；PR #1仍 open Draft |
| Customer-compliant environment | Planned / Inferred | `R0-clean` 尚未在 910C 执行 |
| GLM migration code | Not started | 按用户要求 |
| Performance optimization | Not started | 必须在 correctness/baseline 后 |
| Server / DeepSeek execution | Not authorized / not started | 按用户要求 |

## 已确认的高影响事实

- 官方 A3 CI image 是完整 vllm-ascend runtime；CI 没有卸载/覆盖其 package。
- CI 中 `vllm-ascend` installed/discoverable，但 `VLLM_PLUGINS=fl` 使实际 platform 为 FL。
- 官方当前 910C 成功证据是 Qwen3.6-27B/35B-A3B TP2；GLM、W8A8、MTP、EP、多机、FlagCX均未覆盖。
- FL current main 使用 vLLM 0.20.2；GLM-5.2 官方要求 0.23.0+ 的 IndexShare/MTP reuse。
- 当前 FL Ascend 上 usable MLA、DSA/SFA、wired NPU Indexer、W8A8 Linear、AscendV1 reader 为 Missing。
- 默认通信 backend 是 HCCL；FlagCX optional。
- 当前 FL Ascend 不构建自身 native extension；`VLLM_VENDOR` 必须 unset，设置 `ascend` 会失败。

## 当前阻塞/待决策

1. 客户“FlagOS 原生”是否允许官方 FL 仓库内注明 adapted/copied from vllm-ascend 的历史代码；若也禁止，官方 current main 本身不合规。
2. 客户是否允许中性 `quay.io/ascend/cann:9.0.0-a3-ubuntu22.04-py3.11`，还是只允许 host clean install。
3. A3 现场 Driver/Firmware/ATB exact version 与 topology 尚未审计。
4. GLM vLLM 路线：0.23 最小升级、0.24 dev/未合入 Ascend PR，或 0.20.2 backport。
5. 客户 checkpoint 是 AscendV1 还是 compressed-tensors；manifest、SHA、tensor/scale layout 未提供。
6. legacy repo transfer/new fork 的目标 owner 尚未选择。

## GitHub 状态

- `yanceng305-collab/vllm-plugin-FL` 已是官方 fork network 的 fork；direct parent 是 `xiemingda-1002/vllm-plugin-FL`，network source 是 `flagos-ai/vllm-plugin-FL`。
- PR #1：open Draft；base `ascend-model-migration`，head `audit/glm52-w8a8-stage0-gap`。
- 仅 rename 不会退出 fork network，不能可靠释放同账号重新 fork 的 slot。
- 本轮未 rename、transfer、detach、fork、push、建 branch 或修改 PR。
- 当前文档尚未提交 GitHub：新仓库尚未获确认；提前写 legacy 会违反基线重置约束。确认 owner/迁移方案后再把这些 candidate 文档作为新控制面的首次提交。

## 下一门禁

用户审阅并决定：

- “原生”边界；
- neutral CANN base policy；
- legacy/new fork owner 方案。

三项确认后，才把 `Clean Provenance` task 变成 `ready`；当前不向 DeepSeek 下发。
