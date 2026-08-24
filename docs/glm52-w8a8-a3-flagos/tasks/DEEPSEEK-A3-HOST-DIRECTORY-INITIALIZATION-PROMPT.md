# DeepSeek Prompt — A3 Host Directory Initialization

状态：**Ready after user dispatch；不包含FlagOS/A2实验**

## 目标

Task ID：`A3-HOST-DIR-INIT`。Run ID使用UTC timestamp；Evidence写入`/data/tiankuan/zyg/evidence/A3-HOST-DIR-INIT/<run-id>/`。

在第一台A3 Host初始化`/data/tiankuan/zyg/`的`repos/evidence/artifacts/work/legacy`目录，并建立两个唯一长期Git working tree：

- `repos/vllm-plugin-FL-a3-flagos/`
- `repos/glm52-w8a8-a3-flagos-control/`

## 安全边界

- 不移动、删除、覆盖任何现有内容。
- `/root/vllm-plugin-FL/`保持不动，只登记为legacy/unknown provenance。
- 目标目录已存在时先只读检查；不是预期Git repo或identity不匹配则STOP，不覆盖。
- 不创建container，不访问NPU，不执行A2/FlagOS实验，不安装package。
- Code repo checkout `project/glm52-w8a8-v024`，验证commit=`a9435a34dcd7d0a38e3a853535947371a6c62205`、tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`；Control repo checkout任务开始时最新`origin/main`。
- 两个长期repo都记录remote、branch、HEAD、tree和`git status`。

## PASS / STOP

PASS：目录建立完成；两个repo remote/branch/HEAD/tree可解释且working tree无source-affecting dirty diff；legacy内容未改动。

STOP：目录冲突、Git identity不明、clone/sync失败或需要移动/覆盖现有内容。

Control non-fast-forward不改变本任务结果。允许fetch/rebase安全重试，禁止force push；真实冲突时保留Evidence和本地result commit，将Control Sync标为PENDING。

任务结束后按`REPOSITORY-AND-EVIDENCE-RULES.md`生成immutable result snapshot和一任务三指针；本任务Code change/PR应为`N/A`。
