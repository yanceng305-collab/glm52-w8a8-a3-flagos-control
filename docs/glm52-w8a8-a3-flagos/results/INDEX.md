# Execution Results Index

DeepSeek生成的run snapshot在首次push后不可修改。Codex acceptance只更新本INDEX或`../STATUS.md`，不修改run snapshot。

| Task | Run | Experiment Result | Code pointer | Immutable snapshot | Server Evidence | Control Sync | Codex Acceptance |
|---|---|---|---|---|---|---|---|
| A2-v024 | 20260824T025250Z | STOP at FlagTree gate | `project/glm52-w8a8-v024@a9435a34` / tree`e5e073ed` / no code change | [result](A2-v024/20260824T025250Z.md) | `/data/tiankuan/zyg/evidence-a2-v024-20260824T025250Z` | SYNCED via [e9999d25](https://github.com/yanceng305-collab/glm52-w8a8-a3-flagos-control/commit/e9999d25aea3f3580c1af0bc0d0a5326785762e8) | PENDING；manifest/checksum未索引，且未批准新的FlagTree/Python路线 |

## 状态语义

- Experiment Result：服务器任务本身的`PASS / STOP / PARTIAL / EXPLORATORY`。
- Control Sync：`SYNCED / PENDING`；non-fast-forward或同步冲突不改变Experiment Result。
- Codex Acceptance：`PENDING / ACCEPTED / RETURNED`，由Codex审查后更新。
