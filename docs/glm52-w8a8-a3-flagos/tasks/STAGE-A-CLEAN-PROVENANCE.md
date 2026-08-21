# Stage A Parent Contract — v0.24 Baseline and Carrier Resolution

状态：**Paused for upstream branch migration / Research in progress**
执行边界：不操作服务器、不修改正式代码repo、不生成DeepSeek prompt

## Supersession chain

1. `c70aa4b`的neutral-base-only路线已Superseded。
2. `78b6eb06`的runtime-ownership边界仍有效。
3. `118c314`的v0.20.2 A2 Ready/prompt已因official branch migration暂停，不得下发。
4. 新primary line为official new `main` + vLLM0.24；`v0.2.1@92a6f767...`保留maintenance/reference。

## Current Stage A objective

在服务器执行前完成四个控制面闭环：

```text
Official branch/current-main freeze
  -> non-destructive formal code-repo migration approval
  -> v0.24 A3 carrier/compiler/provider tuple resolution
  -> valid two-logical-device A3 pair contract
  -> A3-CP-A2-v024 task review
  -> new prompt approval
```

## Current artifacts

- Branch/environment research：[`../OFFICIAL-V024-BASELINE-RESEARCH.md`](../OFFICIAL-V024-BASELINE-RESEARCH.md)
- Repository migration proposal：[`../CODE-REPOSITORY-MIGRATION-V024.md`](../CODE-REPOSITORY-MIGRATION-V024.md)
- New A2 draft：[`STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE-DRAFT.md`](STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE-DRAFT.md)
- Historical paused A2：[`STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](STAGE-A2-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)
- Historical paused prompt：[`DEEPSEEK-A3-CP-A2-EXECUTION-PROMPT.md`](DEEPSEEK-A3-CP-A2-EXECUTION-PROMPT.md)

## New A2 Ready requirements

1. 用户批准并执行正式代码repo的非破坏性0.24 branch migration。
2. mutation时重新冻结official main exact SHA/tree。
3. `quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`本机identity可验证；缺失/不确定时STOP且不pull。
4. FlagTree rc1替换carrier triton-ascend的transaction形成single coherent provider；否则STOP。
5. 第一台A3上通过只读拓扑证据确认至少两个logical devices构成valid working pair且完整空闲。
6. FL安装规则冻结为readonly source -> writable staging with `.git` -> exact SHA/tree/clean -> `--no-build-isolation --no-deps -e`。
7. 新A2 task通过用户复核后，才生成新的DeepSeek prompt。

## Unchanged boundaries

- FlagOS execution ownership仍为`PlatformFL -> WorkerFL -> ModelRunnerFL -> Dispatch -> FlagGems/vendor.ascend/Reference -> provider/torch_npu -> CANN`。
- FlagGems preferred而非mandatory；FlagTree是compiler/provider，不是vllm-ascend backend代理。
- FlagCX/FlagScale、graph、MTP、performance、multinode model execution均不进入A2。
- GLM first eager仍必须使用真实W8A8 checkpoint并闭合MLA、DSA/SFA、Indexer、W8A8 Linear/MoE及必要runtime。
- 第二台服务器不用于A2；A2只需第一台上的valid logical-device pair。

## Stop condition

在上述Ready requirements关闭前，不得恢复old prompt、生成new prompt、创建container、pull image或启动Qwen/GLM。
