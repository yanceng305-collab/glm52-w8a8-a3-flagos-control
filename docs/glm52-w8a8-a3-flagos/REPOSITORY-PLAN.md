# 新仓库与 Legacy 保全方案

状态：Superseded by implemented standalone decision；保留为pre-decision历史分析

## 已实施决策

用户已选择“不迁移legacy，创建personal standalone A3代码仓库并配置official upstream”。实施结果和当前权威baseline见[`CODE-REPOSITORY-BASELINE.md`](CODE-REPOSITORY-BASELINE.md)。本文件后续transfer/formal-fork方案不再是当前执行路线；任何legacy mutation仍需新的明确授权。

## 当前事实

- `yanceng305-collab/vllm-plugin-FL` 已是 official fork network 成员：direct parent=`xiemingda-1002/vllm-plugin-FL`，network source=`flagos-ai/vllm-plugin-FL`。
- PR #1 为 open Draft：base=`ascend-model-migration`，head=`audit/glm52-w8a8-stage0-gap`。
- Legacy共有12个可见branch和5个tag；关键branch SHA已在研究记录冻结。
- 仅rename不会离开fork network；GitHub transfer前提也以“目标owner不能已有同一network fork”为约束。因此“同账号只rename再重新fork”高置信不可行，未通过mutation实测。
- Detach/leave network会丢失现有PR/issue/comment等metadata；delete/recreate更直接违反用户禁令，全部拒绝。

GitHub官方行为证据：[rename](https://docs.github.com/en/repositories/creating-and-managing-repositories/renaming-a-repository)、[transfer](https://docs.github.com/en/repositories/creating-and-managing-repositories/transferring-a-repository)、[detach](https://docs.github.com/en/pull-requests/how-tos/work-with-forks/detaching-a-fork)。

## 首选方案

把 legacy fork 连同metadata转移到用户控制、当前不含该fork-network成员的organization/owner，并同时改名为 `vllm-plugin-FL-a2-legacy`；完成post-transfer审计后，再由 `yanceng305-collab` 正式fork `flagos-ai/vllm-plugin-FL`。

### 准备执行的操作（当前未执行）

1. 选择/创建目标owner，并检查repo/Actions/ruleset policy。
2. Preflight导出legacy repo metadata、全部branch/tag SHA、PR #1 URL/state/base/head、Actions secrets/environments/webhooks/packages清单。
3. Transfer legacy，并在transfer时改名为 `vllm-plugin-FL-a2-legacy`。
4. 验证branches/tags/PR #1/history/settings均保留；记录新的canonical legacy/PR URL。
5. 由`yanceng305-collab`正式fork `flagos-ai/vllm-plugin-FL`，只复制/同步操作时official current `main`。
6. 验证新repo direct parent/network source为official；配置`origin=new fork`、`upstream=flagos-ai/vllm-plugin-FL`。
7. 新建`project/glm52-w8a8-a3-flagos-control`，首次提交本轮候选控制面；不导入legacy branch。

### 为什么

- Transfer可保留commit、branch、tag、PR等metadata，同时把existing fork slot移到另一owner。
- New personal fork可获得清晰official upstream关系与独立A3控制面。
- Legacy history与新代码基线物理隔离，避免误用旧A2/vendor环境结论。

### 对 Legacy 的影响

- Branch/tag/history/PR随transfer保留；canonical URL改变。
- GitHub会为旧位置提供redirect；但一旦在原`owner/name`创建新fork，该旧redirect会永久失效。因此必须把新的legacy canonical URL和PR #1 URL写入控制面，不能依赖旧URL。
- Actions secrets/environments/webhooks/packages迁移边界必须逐项验收；当前Unknown。

### 新 Upstream 关系

```text
flagos-ai/vllm-plugin-FL (official upstream)
  -> yanceng305-collab/vllm-plugin-FL (new formal fork, A3 project)

<legacy-owner>/vllm-plugin-FL-a2-legacy
  (preserved history; remains in official fork network; never a new baseline)
```

## 安全替代

1. **最少影响legacy：** 保持legacy完全不动，在另一个owner正式fork official；新项目不在personal namespace。
2. **Fallback：** 在personal account创建standalone `vllm-plugin-FL-a3-flagos`，配置official为upstream；保留旧fork不动。缺点是GitHub UI不显示formal fork relationship。

## 操作前必须由用户确认

- 目标legacy owner/organization与最终URL；
- 是否接受原legacy URL redirect在创建新fork后失效；
- 选择首选transfer方案、另owner新fork，或standalone fallback；
- org/Actions/ruleset/secrets/webhooks/packages的owner与验收人。

未经确认，不执行rename、transfer、fork、push、branch或PR mutation。
