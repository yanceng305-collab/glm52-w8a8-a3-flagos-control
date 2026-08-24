# Repository, Host Working Tree, and Evidence Rules

状态：**Active**
生效日期：2026-08-24

## 职责边界

| 位置 | 责任 | 内容 |
|---|---|---|
| Control repo | Codex控制面；DeepSeek只回传结果 | PLAN、STATUS、DECISIONS、tasks/prompts、Codex acceptance、`results/**` |
| Code repo | 实际代码 | vllm-plugin-FL源码、测试、task branch、commit、PR |
| Server Evidence | 真实服务器证据 | raw logs、report、manifest、checksums；不是长期源码working tree |

Code primary integration branch固定为`project/glm52-w8a8-v024`。代码任务默认从它创建`task/<task-id>-<short-name>`，不得直接开发现有`main`或immutable baseline branch。

## A3 Host目录

```text
/data/tiankuan/zyg/
  repos/
    vllm-plugin-FL-a3-flagos/
    glm52-w8a8-a3-flagos-control/
  evidence/
    <task-id>/<run-id>/
  artifacts/
  work/
    <task-id>/<run-id>/
  legacy/
```

- `repos/`是唯一长期Git working tree。
- `evidence/`中的run关闭后不可修改，也不作为长期源码。
- `artifacts/`保存wheel、bundle、source archive等可复用下载物并记录SHA256。
- `work/`保存可删除的build、editable staging和临时解包目录。
- Container只使用`repos/`中的明确源码，或复制到`work/`后使用。
- `/root/vllm-plugin-FL/`保持不动，视为legacy/unknown provenance；`legacy/`只保存索引/路径登记，不复制旧源码。
- Run `A3-HOST-DIR-INIT/20260824T030000Z`已偏离上述规则并创建`legacy/root-vllm-plugin-FL/`副本；原目录未改。现有副本保持不动，未经独立任务批准不得删除、覆盖、继续同步或作为正式源码。

## Codex与DeepSeek写入规则

Codex：

- 写Control repo并负责PLAN、STATUS、DECISIONS、task/prompt与acceptance。
- 每次修改前同步GitHub最新`main`；验收后commit并push。
- 默认不修改Code repo，除非用户明确要求。

DeepSeek：

- 执行服务器实验、实际代码修改、测试、Evidence和PASS/STOP事实。
- 代码修改只提交到Code task branch并push；需要合并时创建base=`project/glm52-w8a8-v024`的PR。
- 默认不修改Control的PLAN、DECISIONS、STATUS、task或prompt。
- 每个run结束后只新增`results/<task-id>/<run-id>.md`并更新`results/INDEX.md`。
- Run snapshot首次push后不可由DeepSeek或Codex修改；Codex acceptance只写`results/INDEX.md`或`STATUS.md`。

## Control同步失败

- Non-fast-forward不改变服务器实验本身的PASS/STOP/PARTIAL状态。
- DeepSeek允许`fetch`后安全rebase并重试push；禁止force push。
- 若存在真实冲突，保留本地result commit与Server Evidence，把Control Sync标记为`PENDING`并停止Control写入；由Codex后续接管。
- Control Sync状态与Experiment Result状态必须分开记录。

## Working tree与正式Evidence

- 开发期间允许exploratory dirty working tree。
- 正式PASS/STOP Evidence必须绑定Code repo exact repo、branch、commit和tree，并证明实际执行源码与该Git tree一致。
- Source-affecting dirty diff未提交时，该run只能标为`EXPLORATORY`，不能作为正式验收Evidence。
- 无代码修改的任务仍记录Code base branch、commit和tree，并把code change/PR写为`N/A`。

## 一任务三指针

每个run必须记录：

1. Code：repo + branch + commit + tree + PR（无代码变化时明确`N/A`）。
2. Control：immutable result snapshot + DeepSeek result commit + Codex acceptance状态/commit。
3. Evidence：server absolute path + manifest/checksum。

三指针不完整时，任务不能形成正式acceptance；聊天记录不作为正式指针。
