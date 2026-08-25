# Codex2 Prompt — CANARY-V024-QWEN36-27B-TP2

状态：**SUPERSEDED / CANCELLED UNEXECUTED by User Decision；historical prompt only；never dispatched。不得下发或执行；不得通过填写旧占位符恢复。**

先同步并读取最新正式Control `AGENTS.md`、`README.md`、`STATUS.md`，然后只执行合同：

[`CANARY-V024-QWEN36-27B-TP2.md`](CANARY-V024-QWEN36-27B-TP2.md)

## User-confirmed facts

- ModelScope repo/revision：`Qwen/Qwen3.6-27B@cea40373b9214dd387123e68841890af30dcd469`
- Official ModelScope：<https://www.modelscope.cn/models/Qwen/Qwen3.6-27B>（只作identity reference，不得由Codex2下载）
- Exact local model path：`<USER MUST FILL>`
- Authorized 910C logical devices：`<USER MUST FILL OR AUTHORIZE TASK-SAFE SELECTION>`
- Dispatch：`<USER MUST EXPLICITLY CONFIRM>`

唯一目标：在accepted v0.24 snapshot和exact clean FL baseline上，对上述本地完整BF16 checkpoint完成TP2/HCCL、text-only、eager offline deterministic smoke，并证明live：

`PlatformFL → WorkerFL → ModelRunnerFL → FlagOS Dispatch → Ascend NPU`

否则在第一个可归因blocker处STOP。

不下载、修复、转换或替换模型；不修改runtime/dependency/FL/Code；不创建Code branch/PR；不继续到GLM、benchmark/profile/optimization；offline未PASS不得serving。普通安全检查、运行和排障方式由你在合同边界内选择。

完成后保存合同要求的Evidence与三指针，生成immutable result并更新`results/INDEX.md`；FL无修改时Code PR=`N/A`。
