# Stage A Parent Contract — v0.24 A2 Bring-up

状态：**Active；A2 Ready — ordered environment preflight then shared-NPU tiny smoke**
本轮Codex边界：代码repo/control操作已授权并完成；不操作A3服务器

## Completed prerequisites

- Official freeze：`main@a9435a34...`/tree`e5e073ed...`；`v0.2.1@92a6f767...`。
- Formal repo migration：三个branch已精确创建；existing main和legacy零变化。
- Official documented release tag仍为`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`，但当前无可用artifact；A2 provisional carrier固定为`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`，身份仅为official `releases/v0.24.0rc` A3 nightly。
- Carrier内部runtime tuple不预设，全部以container preflight为准。
- A3 device contract：环境preflight不映射NPU；其PASS后允许共享NPU 12+13执行极小`torch_npu` tensor和FlagOS Dispatch operator smoke。
- Compiler contract：先按实际carrier inventory审计provider，再在disposable container内用package manager替换为exact FlagTree并验证single coherent provider；失败即STOP。
- FlagGems：v5.3.4 exact source，组件必须安装/确认，但operator不强制选FlagGems。
- FL install：readonly project branch -> writable `.git` staging -> exact SHA/tree/clean -> `--no-build-isolation --no-deps -e`。
- Formal FL source可direct clone；服务器GitHub不可达时允许expected SHA256校验过的Git bundle relay，最终验证branch/HEAD/tree/clean。
- Restricted network：carrier只接受exact digest；网络仅允许FlagTree official resource index和official FlagGems tag。

## Ready task and prompt

- [`STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)
- [`DEEPSEEK-A3-CP-A2-V024-EXECUTION-PROMPT.md`](DEEPSEEK-A3-CP-A2-V024-EXECUTION-PROMPT.md)

Historical v0.20.2 task/prompt继续Paused，不得使用。

## A2 execution chain

```text
Phase A: exact carrier without NPU mapping
  -> carrier runtime inventory + bundle/Git identity
  -> FlagTree/FlagGems/FL preparation
  -> static validation
  -> environment gate PASS
  -> Phase B: new restricted NPU-enabled disposable container
  -> share only NPU 12+13
  -> replay Phase A environment
  -> tiny torch_npu tensor
  -> one tiny FlagOS Dispatch operator
  -> STOP for Codex review
```

## Non-goals

- Qwen/GLM模型与服务；
- HCCL/TP、KV cache、完整WorkerFL/ModelRunnerFL runtime和大tensor；
- MLA、DSA/SFA、Indexer、W8A8、完整attention/MoE；
- benchmark/profile/performance；
- 第二台服务器；
- Host Driver/Firmware/CANN/network修改；
- 下一Stage自动执行。

## Stage exit

Phase A任一门禁失败即STOP；全部PASS后无需再次确认，直接进入受限Phase B。共享NPU状态恶化或可能干扰现有任务时立即STOP。A2只有环境preflight、tiny tensor与Dispatch operator全部通过后才完成。
