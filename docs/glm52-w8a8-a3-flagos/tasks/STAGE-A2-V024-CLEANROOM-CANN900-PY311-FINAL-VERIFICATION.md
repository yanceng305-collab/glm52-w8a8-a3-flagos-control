# A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION

状态：**Ready after User dispatch**

执行代理：Codex2

执行对象：现有正式container `flagos-cann900-py311-test`

## 目标

这是`A2-V024-CLEANROOM-CANN900-PY311`的**最后一次最小Evidence follow-up**。只补两类Acceptance证据：

1. 对现有container做有边界的clean-room / no-copy独立核查；
2. 对四项既有tiny smoke补显式device、backend/provider、hard assertion、result与no-silent-CPU-fallback证据。

同一follow-up只读恢复当前container/image metadata，输出可重建的container配置和runtime重放入口。Original run、Codex2 supplemental Evidence与本task Evidence联合用于Codex1最终Acceptance。

## 正式输入

- Original task：[`STAGE-A2-V024-CLEANROOM-CANN900-PY311.md`](STAGE-A2-V024-CLEANROOM-CANN900-PY311.md)
- Immutable result：[`../results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md`](../results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z.md)
- Supplemental pointer：[`../results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z-evidence-supplement-codex2.md`](../results/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z-evidence-supplement-codex2.md)
- Reconstruction baseline：[`../A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md`](../A2-V024-CLEANROOM-CANN900-PY311-RECONSTRUCTION.md)
- Time boundary：[`../DECISIONS.md`](../DECISIONS.md) D-076
- Original run：`20260824T080753Z`
- Existing container：`flagos-cann900-py311-test`
- Frozen Code：`yanceng305-collab/vllm-plugin-FL-a3-flagos` / `project/glm52-w8a8-v024@a9435a34dcd7d0a38e3a853535947371a6c62205` / tree `e5e073edf4b65c053e954d78d20365aab0e1f46b`
- New Evidence：`/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION/<run-id>/`

User-confirmed base pull command：

```bash
docker pull swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:9.0.0-a3-ubuntu22.04-py3.11-devel
```

本task不得执行该pull；它只作为reconstruction的已确认输入。

## 不可变边界

- 不重建、recreate或替换container；不重装、升级、卸载或patch任何package。
- 不修改现有runtime、FL source、Code repo、原始Evidence、supplemental Evidence或旧immutable result。
- 不应用/回滚两个third-party patch，不创建FL branch/commit/PR；Code PR=`N/A`。
- 除四项tiny smoke外不重复实验；禁止模型加载、serve、KV cache、HCCL/TP/collective、benchmark、profile和大tensor。
- Host/现有任务只读；共享NPU状态恶化、可能干扰其他任务或需要runtime mutation时立即STOP。
- 所有新输出写入本task的新Evidence目录；关闭后生成manifest、SHA256和成功的checksum verification log。

## A. Clean-room / no-copy最小核查

目标不是补造完美的历史操作审计，而是用当前可验证事实建立正向provenance并检查是否存在实质反证。至少保存：

1. container与base image identity：name/ID/created/state、configured image、immutable image ID、RepoDigest、architecture和created metadata，并核对frozen digest `sha256:5f20011b2c5509ca4716393e66fc7aa07016629bce36a7f6c32c1bf31f30433f`。
2. 完整mount/bind/volume与关键Docker runtime配置；明确检查是否挂载上一轮copied-runtime的Evidence/Work、Qwen-derived runtime、foreign Python/site-packages或其他container root。
3. Python/package/import provenance：Python prefix/path，torch、torch_npu、vLLM、`triton`/FlagTree、FlagGems、`vllm_fl`的distribution、version、module realpath、direct-url/RECORD或等价metadata；检查路径是否指向正式repo、当前container或已记录artifact。
4. FL identity：branch/HEAD/tree、clean status、editable import realpath；不得把generated metadata误判为授权FL修改。
5. Provider ownership：FlagTree拥有active `triton`namespace与Ascend backend；独立`triton`/`triton-ascend`distribution、shadowing或mixed-provider文件不得存在。
6. 将现场identity与original/supplemental manifest中的已保存artifact、patch和install provenance交叉映射。

如果以上核查没有发现正向copy/reuse痕迹，但当前状态无法绝对证明历史上从未发生`docker cp`或同类操作，应明确写为`residual risk / evidence debt`，不能仅因此STOP或派生新A2验证。

## B. 四项tiny smoke显式验证

只重复原run的四类最小路径。推荐使用一个保存并checksum的harness，但普通实现方法由Codex2决定。每一项必须在任何`.cpu()`或host conversion之前记录并hard assert：

| 必存字段 | 要求 |
|---|---|
| device mapping | container-visible device及其到获授权physical NPU 12/13的映射 |
| input/output device | 调用前input与调用后output的完整device；output须保持NPU |
| backend/provider | 实际选择的implementation/backend/provider及resolved module path |
| no-fallback | hard assertion证明output仍为`npu`且走预期NPU路径 |
| correctness | tiny expected value/tolerance；CPU只可在NPU断言后作为oracle |
| synchronization | 显式NPU同步，使异步错误成为失败 |
| result | PASS/FAIL、完整stdout/stderr与numeric process exit status |

四项范围：

1. `torch_npu` tiny tensor arithmetic；可在同一case覆盖`npu:0`与`npu:1`。
2. 原FlagTree tiny add kernel；记录`triton.backends.ascend`origin与FlagTree single-provider identity。
3. FlagGems direct `gems_silu_and_mul`；记录实际Ascend vendor/implementation/module。
4. `VLLM_PLUGINS=fl`下的FL/Dispatch `silu_and_mul`；记录`PlatformFL npu`与Dispatch-selected implementation/backend/module。

与本算子无关的warning不单独构成失败；必须按所测路径的实际assertion判断。

## A2. Container/environment reconstruction恢复（属于A类provenance，不是第三类验证）

不得猜测实际`docker run`。从当前server/container metadata和现有Evidence恢复并保存：

- raw、必要脱敏的container/image inspect；
- normalized reconstruction参数表；
- `rebuild-container.sh`（或等价脚本），固定image digest和container name，并覆盖实际device、mount/volume、network/ports、shm/IPC、runtime、privileged/capability/security、ulimit/resource、user/workdir、environment、entrypoint/cmd/restart等非默认关键配置；
- 实际container进入方式和必要environment bootstrap；
- `rebuild-runtime.sh`或等价ordered manifest，绑定torch/torch_npu artifact、vLLM empty、FlagTree、FlagGems、两个patch replay和FL exact Git identity；
- 每个值的Evidence来源；无法现场确认的值必须标`Unknown / not recoverable`，不得编造。

本task只静态检查脚本语法/内部一致性，不执行pull、docker run或runtime rebuild。可配置host-local值与项目frozen值应分开。

## 最小Evidence集合

- `followup-report.md`：两类验证、reconstruction交付、PASS/STOP、residual risk；
- clean-room/no-copy structured report及raw supporting log；
- tiny-smoke harness、stdout/stderr、numeric exit-status记录；
- raw/redacted inspect、normalized Docker配置、container entry说明；
- `rebuild-container.sh`与runtime replay/lock；
- original run + supplemental + follow-up联合指针；
- 本目录manifest、SHA256和成功verification log。

结束后新增`results/A2-V024-CLEANROOM-CANN900-PY311-FINAL-VERIFICATION/<run-id>.md`并更新`results/INDEX.md`；不得修改任何既有result snapshot。

## PASS

Codex2可报告PASS，当且仅当：

- existing container/base/runtime/Code/FL/provider核查一致，未发现旧runtime copy/reuse或其他实质反证；
- 四项tiny smoke全部显式证明NPU input/output、实际backend/provider、no fallback、correctness、同步与exit=0；
- runtime与FL保持未修改；
- reconstruction参数和脚本已从现场恢复，Unknown均明确标注并有边界说明；
- 新Evidence完整、checksum通过，三套Evidence联合指针完整。

如果只剩无法恢复的旧命令exit code、无法做到完美历史negative audit或类似Evidence格式债务，报告PASS并列为residual risk/evidence debt，不得扩张为新的A2验证。

## STOP

发现以下任一实质性技术反证时STOP并保存最小证据：

- base image/runtime identity错误或关键tuple漂移；
- 存在使用上一轮container/runtime、foreign Python/site-packages或普通目录复制的正向证据；
- Code identity错误、FL dirty或有未记录修改；
- mixed compiler/provider ownership；
- 任一case实际CPU fallback、选择错误backend、NPU执行/同步/correctness失败或无法在NPU完成；
- 为通过验证必须修改runtime、FL或closed Evidence。

不得依据D-076把真实失败强行判PASS。若existing container已不存在/不可读，STOP；不得自行重建后代替本task。

## 时间与Acceptance边界

D-076约束：这是A2 environment gate最后一次Evidence follow-up。若没有上述实质反证，Codex1在联合审查后应把不可恢复的历史审计缺口登记为residual risk/evidence debt并结束A2验证循环；Acceptance仍只覆盖clean-room runtime/compiler/Dispatch tiny-smoke gate，不降低后续GLM模型加载、W8A8 correctness、E2E、serve或性能标准。
