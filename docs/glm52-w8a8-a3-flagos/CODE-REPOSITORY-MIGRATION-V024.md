# Formal Code Repository Migration Proposal — vLLM 0.24 Line

状态：**Implemented / PASS**
目标repo：`yanceng305-collab/vllm-plugin-FL-a3-flagos`

## Current facts

- Repo为standalone，default branch `main`。
- Migration前只有`main@92a6f767...`；migration后为下述四个exact branches，仍无tag/PR。
- 该SHA现在是official `v0.2.1` / vLLM0.20.2 maintenance/reference HEAD。
- 当前repo没有PR、没有tag、`main`无branch protection。
- Mutation前已重新读取official new `main`，本轮freeze为`a9435a34...`/tree`e5e073ed...`。
- official `v0.2.1`与new `main`已diverged，不能fast-forward。
- legacy repo `yanceng305-collab/vllm-plugin-FL`及PR #1完全不在本方案mutation范围。

## Execution result

| Branch | Exact SHA | Tree |
|---|---|---|
| existing `main` | `92a6f7670465922c60e88f06787b8f0923e761f3` | `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7` |
| `baseline/official-v0.2.1-vllm0.20.2` | `92a6f7670465922c60e88f06787b8f0923e761f3` | `e610bc5828b4a4a54a8f55429b40500ff4f5a0a7` |
| `baseline/official-main-vllm0.24-20260821-a9435a3` | `a9435a34dcd7d0a38e3a853535947371a6c62205` | `e5e073edf4b65c053e954d78d20365aab0e1f46b` |
| `project/glm52-w8a8-v024` | `a9435a34dcd7d0a38e3a853535947371a6c62205` | `e5e073edf4b65c053e954d78d20365aab0e1f46b` |

- Atomic push创建三个branch；没有force、merge、rebase或cherry-pick。
- Existing `main`和GitHub default branch均保持不变。
- 按用户要求，本轮未配置branch protection或复杂PR规则。
- Legacy 12 branches hash前后=`70f9d104d86bb7a35e109b418c59908485ab524e4987402bf21a62fdd102bb3c`。
- Legacy 5 tags hash前后=`1c17d3cd71d618d2d2a3f7a73608d5b78c8feabbcee8f1aa1d5104b940e88355`。
- Legacy PR #1 hash前后=`b77735cc490c530c36179b5ddba283a27caebbc003d1c99857aba790a2e84a41`；仍OPEN/Draft，base/head SHA不变。

## Implemented least-risk same-repository layout

不修改现有`main`，通过新增可审计branch把维护线、immutable upstream snapshot与项目integration分开：

```text
existing main
  92a6f767...  (保持不变；v0.2.1 maintenance/reference)

baseline/official-v0.2.1-vllm0.20.2
  92a6f767...  (新增immutable anchor)

baseline/official-main-vllm0.24-<freeze-date>-<short-sha>
  <mutation时重新冻结的official main exact SHA>

project/glm52-w8a8-v024
  从上述0.24 baseline exact SHA创建；作为primary integration/PR base

capability/<mla|dsa|indexer|w8a8|...>
  从同一0.24 baseline或control批准的project head派生；独立Draft PR
```

### Executed operation sequence

1. 重新读取official default branch、`main` HEAD/tree与`v0.2.1` HEAD/tree；若与control research freeze不同，先更新control，不执行mutation。
2. 只读快照existing repo branches/tags/PR/default/protection与remote heads。
3. 从existing `main@92a6f767...`新增`baseline/official-v0.2.1-vllm0.20.2`。
4. 从official exact new-main SHA直接新增`baseline/official-main-vllm0.24-<date>-<sha>`；不得merge或cherry-pick整条0.24 history到old main。
5. 从0.24 immutable baseline新增`project/glm52-w8a8-v024`。
6. existing `main`保持commit、tree和名称不变，不删除、不force-push、不merge divergent histories。
7. 记录origin/upstream、branch SHA/tree和可达性；验证legacy repo/PR #1零变化。
8. Default branch/protection治理明确后置，不阻塞A2。

## Why this is recommended

- 不重写或覆盖existing `main`；
- `92a6f767...`保留双重可追溯锚点；
- new main exact upstream snapshot不可变；
- primary integration branch名称明确，不把baseline branch用于直接开发；
- capability PR拥有稳定base和清晰回滚点；
- 无需创建第三个代码repo，也无需formal fork/detach。

## Effects and risks

| 操作 | 影响 | 风险控制 |
|---|---|---|
| 新增baseline branches | 不改变existing refs | exact SHA/tree验收 |
| 新增project branch | 新primary开发线 | 只从0.24 snapshot创建 |
| Default/protection未改 | 不影响当前bring-up | 后续按需治理 |
| 不merge两条diverged history | 避免混入v0.2.1-only commits | capability按新main重新gap确认 |

## Alternative: separate v0.24 code repository

可另建`vllm-plugin-FL-a3-flagos-v024`并精确复制official new main。它对existing repo零影响，但会分裂issue/PR/remote管理与项目身份。除非客户要求new repo的`main`必须就是0.24 baseline，否则不推荐。

## Explicitly forbidden

- force-push new main到existing `main`；
- 删除或rename existing `main`；
- merge/rebase divergent v0.2.1和new main以制造伪升级；
- 把旧A2/capability代码cherry-pick到0.24 baseline而不重新审查；
- 修改legacy repo、PR #1或official upstream；
- 未经用户批准执行本提案。

## Acceptance

Repository migration已完成并验收；后续capability branch以`project/glm52-w8a8-v024`为integration base，并受control task约束。
