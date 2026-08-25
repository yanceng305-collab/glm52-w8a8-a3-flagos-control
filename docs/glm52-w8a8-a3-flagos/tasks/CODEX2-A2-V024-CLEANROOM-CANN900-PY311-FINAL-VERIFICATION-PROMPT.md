# Codex2 Prompt — A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION

你是Codex2，服务器主执行代理。

先同步并读取最新正式Control repo的`AGENTS.md`、`README.md`、`STATUS.md`，再执行唯一合同：

[`STAGE-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION.md`](STAGE-A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION.md)

## User-confirmed facts

- 现有正式container：`flagos-cann900-py311-test`
- 已确认base pull命令：
  `docker pull swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:9.0.0-a3-ubuntu22.04-py3.11-devel`

本task不得执行pull或重建container；pull命令只用于reconstruction交付。

## 本次唯一目标

在现有container上完成最后一次最小A2验证：

1. 有边界地核查clean-room/no-copy provenance；
2. 为torch_npu、FlagTree kernel、FlagGems direct、FL/Dispatch四项tiny smoke补齐显式device/backend/no-fallback/assertion/result；
3. 作为第1类provenance核查的长期交付，通过`docker inspect`、image/container metadata和既有Evidence恢复实际container配置，返回checksum-bound的可重建container/runtime脚本；无法确认项明确标Unknown。

普通只读调查、harness与排障方法由你在合同边界内自主选择。不要重复`AGENTS.md`长期规则，也不要扩张为完整runtime provenance或模型实验。

## 硬边界

- 不重建/recreate、重装、升级、卸载、patch或修改现有runtime；
- 不修改FL、Code repo、old result、original/supplemental Evidence；
- 新Evidence写入合同规定的新task/run目录；
- 除四项tiny smoke外不运行其他实验；
- 不开始GLM适配、模型加载、serve、collective、benchmark或profile；
- 发现实质技术反证按合同STOP，不得用D-076强行PASS；
- 没有实质反证时，不因无法补出完美历史negative audit或旧无关exit code而衍生新A2验证。

结束后生成新immutable result并更新`results/INDEX.md`。Original run、Codex2 supplement与本follow-up Evidence必须形成联合指针；FL无修改时Code PR=`N/A`。
