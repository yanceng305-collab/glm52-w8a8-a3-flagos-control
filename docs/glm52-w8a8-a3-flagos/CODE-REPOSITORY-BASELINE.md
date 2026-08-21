# 正式A3代码仓库基线

记录日期：2026-08-21
冻结时间：2026-08-21T15:18:31.5529156+08:00
建仓授权所依据的control baseline：`ea59bdacf8a894ae8e5bfab61921132cb863c6bb`

## 仓库身份

| 字段 | 冻结值 |
|---|---|
| 正式A3代码仓库 | [yanceng305-collab/vllm-plugin-FL-a3-flagos](https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos) |
| Official repository | [flagos-ai/vllm-plugin-FL](https://github.com/flagos-ai/vllm-plugin-FL) |
| 仓库类型 | Standalone；`isFork=false`；不是formal fork |
| 默认分支 | `main` |
| Official frozen main commit | `92a6f7670465922c60e88f06787b8f0923e761f3` |
| New repository main commit | `92a6f7670465922c60e88f06787b8f0923e761f3` |
| Official frozen tree | `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7` |
| New repository main tree | `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7` |
| Commit equality | PASS |
| Tree equality | PASS |

仓库创建时未初始化README、LICENSE、gitignore或任何额外commit；新`main`直接指向冻结official commit。

## Remote关系

本地正式代码clone固定为：

```text
origin   = https://github.com/yanceng305-collab/vllm-plugin-FL-a3-flagos.git
upstream = https://github.com/flagos-ai/vllm-plugin-FL.git
```

GitHub服务端没有fork parent/source关系；`upstream`是本项目记录和本地Git配置中的权威来源。

## Baseline纯净度验收

- 新仓库只有`main`一个branch；
- 新仓库tag数量为0；
- 未导入legacy branch、tag或migration ref；
- Legacy-only commits `82f3e718...`、`637a549...`、`f8627fcb...`、`4d4645b3...`在新standalone仓库不可达；
- 本地工作区干净，HEAD/tree与remote `main`一致。

## Legacy零变化验收

Legacy仓库：[yanceng305-collab/vllm-plugin-FL](https://github.com/yanceng305-collab/vllm-plugin-FL)。

| 对象 | 前后验收 |
|---|---|
| Repository ID / fork parent / source / settings字段 | 不变 |
| 12个branches及SHA | 哈希一致：`a99a62568a83fdd2535f8968763090a2997a813ee2ca76da604ab614111a99ec` |
| 5个tags及SHA | 哈希一致：`e0b7ee12e430fb7a496e275e170890f9543582a12d822fefe6f80c56621a088c` |
| Draft PR #1 | OPEN/Draft、base/head/head SHA、timestamps均不变；哈希`c7f4ccc0aa8cfee894563be967dec31363df5dcc994bec4c9a37861c07a6846a` |

PR #1仍为：[Freeze W8A8 Ascend gap and eager repro](https://github.com/yanceng305-collab/vllm-plugin-FL/pull/1)，base=`ascend-model-migration`，head=`audit/glm52-w8a8-stage0-gap`。

## Main使用规则

- `main`只表示经control repo批准的official frozen baseline；禁止直接开发；
- MLA、DSA/SFA、Indexer、W8A8等capability原则上使用独立branch和独立Draft PR；
- 不得从legacy创建branch、merge、cherry-pick或导入tag；
- 所有实现受control repo最新`PLAN.md`、`DECISIONS.md`和task contract约束。

## Upstream同步策略

1. 不自动同步official，不使用GitHub fork sync按钮；
2. 每次同步先由control repo形成明确决策，记录old baseline、candidate official SHA/tree和compatibility影响；
3. 只接受指向经过批准的official `main` commit；
4. 新仓库`main`只允许fast-forward到该精确official commit，不创建merge/squash/rebase baseline commit，不force-push；
5. 同步后重新验证commit/tree相等、branch/tag纯净度和mandatory tests；
6. feature branch需要rebase/重建时单独治理，不把实验历史写入`main`。

## 当前停止点

正式代码仓库落地已完成。尚未执行A2 FL-only Environment Smoke、Post-Eager Runtime Provenance Audit、服务器操作、DeepSeek任务、GLM适配、mandatory capability实现或性能测试。
