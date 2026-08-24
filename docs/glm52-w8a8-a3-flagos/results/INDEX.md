# Execution Results Index

DeepSeek生成的run snapshot在首次push后不可修改。Codex acceptance只更新本INDEX或`../STATUS.md`，不修改run snapshot。

| Task | Run | Experiment Result | Code pointer | Immutable snapshot | Server Evidence | Control Sync | Codex Acceptance |
|---|---|---|---|---|---|---|---|
| A2-V024-PY311-FIRST-FLAGTREE-ENV | 20260824T065902Z | PASS | `project/glm52-w8a8-v024@a9435a34` / tree`e5e073ed` / no code change | [result](A2-V024-PY311-FIRST-FLAGTREE-ENV/20260824T065902Z.md) | `/data/tiankuan/zyg/evidence/A2-V024-PY311-FIRST-FLAGTREE-ENV/20260824T065902Z` | SYNCED via [8bc491d](https://github.com/yanceng305-collab/glm52-w8a8-a3-flagos-control/commit/8bc491dba60c60c745246850a20315a0667a77a9) | PENDING |
| A3-HOST-DIR-INIT | 20260824T030000Z | PASS | `project/glm52-w8a8-v024@a9435a34` / tree`e5e073ed` / no code change | [result](A3-HOST-DIR-INIT/20260824T030000Z.md) | `/data/tiankuan/zyg/evidence/A3-HOST-DIR-INIT/20260824T030000Z` | SYNCED via [57c164e](https://github.com/yanceng305-collab/glm52-w8a8-a3-flagos-control/commit/57c164e97168e3578d647097dd564d8e58505f93) | **ACCEPTED**；deviation：旧源码被复制到`legacy/root-vllm-plugin-FL/`，原目录未改；现有副本不处理，future legacy只登记不复制 |
| A2-v024 | 20260824T025250Z | STOP at FlagTree gate | `project/glm52-w8a8-v024@a9435a34` / tree`e5e073ed` / no code change | [result](A2-v024/20260824T025250Z.md) | `/data/tiankuan/zyg/evidence-a2-v024-20260824T025250Z` | SYNCED via [e9999d25](https://github.com/yanceng305-collab/glm52-w8a8-a3-flagos-control/commit/e9999d25aea3f3580c1af0bc0d0a5326785762e8) | PENDING；manifest/checksum未索引，且未批准新的FlagTree/Python路线 |

## 状态语义

- Experiment Result：服务器任务本身的`PASS / STOP / PARTIAL / EXPLORATORY`。
- Control Sync：`SYNCED / PENDING`；non-fast-forward或同步冲突不改变Experiment Result。
- Codex Acceptance：`PENDING / ACCEPTED / RETURNED`，由Codex审查后更新。
