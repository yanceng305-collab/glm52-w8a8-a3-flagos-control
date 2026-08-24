# Stage A Parent Contract — v0.24 A2 Bring-up

状态：**Active；Phase A Ready；Phase B Waiting for valid free pair**
本轮Codex边界：代码repo/control操作已授权并完成；不操作A3服务器

## Completed prerequisites

- Official freeze：`main@a9435a34...`/tree`e5e073ed...`；`v0.2.1@92a6f767...`。
- Formal repo migration：三个branch已精确创建；existing main和legacy零变化。
- Primary carrier：`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`。
- A3 device contract：Phase A不需要NPU；Phase B至少需要一个valid、完整空闲的two-logical-device pair。
- Compiler contract：在disposable container内用package manager把carrier triton-ascend替换为exact FlagTree，并验证single coherent provider；失败即STOP。
- FlagGems：v5.3.4 exact source，组件必须安装/确认，但operator不强制选FlagGems。
- FL install：readonly project branch -> writable `.git` staging -> exact SHA/tree/clean -> `--no-build-isolation --no-deps -e`。
- Restricted network：只允许exact carrier、FlagTree official resource index和official FlagGems tag。

## Ready task and prompt

- [`STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`](STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md)
- [`DEEPSEEK-A3-CP-A2-V024-PHASE-A-EXECUTION-PROMPT.md`](DEEPSEEK-A3-CP-A2-V024-PHASE-A-EXECUTION-PROMPT.md)

Historical v0.20.2 task/prompt继续Paused，不得使用。

## A2 execution chain

```text
Phase A: exact carrier without NPU mapping
  -> package/compiler/FlagGems/FL preparation
  -> static validation
  -> STOP + Phase A Evidence

Wait for valid free pair
  -> Phase B: new NPU-enabled disposable container
  -> replay Phase A environment
  -> torch/torch-npu/pair smoke
  -> PlatformFL/WorkerFL/ModelRunnerFL/Dispatch
  -> one NPU-resident synthetic operator
  -> STOP for Codex review
```

## Non-goals

- Qwen/GLM模型与服务；
- MLA、DSA/SFA、Indexer、W8A8、完整attention/MoE；
- benchmark/profile/performance；
- 第二台服务器；
- Host Driver/Firmware/CANN/network修改；
- 下一Stage自动执行。

## Stage exit

Phase A完成后必须先由Codex验收；不得因没有free pair阻止Phase A，也不得从Phase A直接进入Phase B。A2整体只有Phase B runtime smoke通过后才完成。
