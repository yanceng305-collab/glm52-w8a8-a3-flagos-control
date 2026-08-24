# DeepSeek执行提示词 — A3-CP-A2-v024

状态：**Ready**

唯一合同：`docs/glm52-w8a8-a3-flagos/tasks/STAGE-A2-V024-OFFICIAL-CARRIER-FL-ONLY-ENVIRONMENT-SMOKE.md`。

## 目标

在第一台A3服务器完成：

```text
exact carrier digest + Git bundle identity preflight
  -> FlagTree
  -> FlagGems v5.3.4
  -> FL project/glm52-w8a8-v024
  -> tiny torch_npu tensor on shared NPU 12+13
  -> one tiny FlagOS Dispatch operator
  -> PASS or STOP
```

环境门禁全部PASS后直接继续tiny NPU smoke，不等待再次确认；任一环境门禁失败不得进入NPU步骤。

## 已确认事实

- Official文档定义`quay.io/ascend/vllm-ascend:v0.24.0rc1-a3`，但当前没有可用release artifact。
- 唯一授权carrier是`quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585`。
- 该artifact只能称为official `releases/v0.24.0rc` A3 nightly，不得称为rc1 release image。
- 不预设carrier内部OS/Python/CANN、torch/torch-npu、vLLM、triton/triton-ascend、transformers或其他runtime版本；以container preflight实测为准。
- FL formal identity必须为branch=`project/glm52-w8a8-v024`、HEAD=`a9435a34dcd7d0a38e3a853535947371a6c62205`、tree=`e5e073edf4b65c053e954d78d20365aab0e1f46b`、worktree clean。
- 服务器GitHub不可达时，允许使用WORKDIR内本次预置的唯一Git bundle及其expected SHA256 sidecar；缺失、出现多个候选或校验失败即STOP。校验后从bundle clone，保留`.git`。
- FlagTree固定`0.6.1rc1+ascend3.5`，仅来自FlagOS official resource index；FlagGems固定v5.3.4、commit`f7c55cb2fe1fb90fc713eafa6f63d7cbb73453a9`、tree`87e4e1e98c80dfd31d923bd726795f385aa28ffd`。
- NPU 12+13已有其他任务但允许共享，仅用于本任务的极小tensor和Dispatch operator smoke。

## 安全边界

- 新建唯一Evidence目录；记录命令、退出码、raw stdout/stderr、checksums、image/container identity、实际package inventory和pre/post diff。
- Carrier digest缺失或不匹配即STOP；禁止使用release tag、mutable nightly tag或其他image fallback。
- `/root/vllm-plugin-FL/`不是Git repo，保持不动；formal source只能从校验过的bundle或唯一formal repo取得，clone到Evidence新目录。
- 在environment container内按合同仅用package manager卸载`vllm-ascend`，并由fresh Python process完成distribution、`find_spec`和entry-point negative check；不得手工删文件制造PASS。
- FL安装前后都验证branch/HEAD/tree/clean；只在container writable `.git` staging中执行`--no-build-isolation --no-deps -e`，不得回写formal source。
- 先按实际inventory审计compiler distributions/RECORD/module/native ownership，再用package manager完成FlagTree replacement；不得预设carrier含`triton-ascend==3.2.1`，不得手工删除package文件，最终必须是single coherent FlagTree provider。
- FlagGems仅使用current container Python、exact source、`--no-build-isolation --no-deps`；禁止`setup.sh`、`flaggems-setup`、bootstrap、独立环境和核心runtime升级。
- 环境PASS后新建受限NPU container，只暴露12+13及必要Driver/DCMI资源。开始和结束都记录现有进程、显存与AICore状态；状态恶化或可能干扰现有任务立即STOP。
- NPU步骤只允许极小`torch_npu` tensor和一个小shape、NPU-resident FlagOS Dispatch operator，并记录selected impl origin、输入输出device、结果及无silent CPU fallback。
- 禁止模型加载、vLLM serve、HCCL/TP、KV cache、完整`WorkerFL`/`ModelRunnerFL` runtime、benchmark/profile、大tensor、Qwen/GLM、MLA/DSA/Indexer/W8A8、完整attention/MoE，以及修改Host、control或正式repo。

## PASS / STOP

只有carrier digest、bundle SHA256、FL Git identity、实际runtime inventory、vllm-ascend negative check、single coherent FlagTree、FlagGems、FL静态origin、tiny torch_npu和Dispatch operator全部通过，才报告`A2-v024 PASS`。

任一identity不匹配、需要改变核心runtime、provider ownership混合、缺build requirement需扩大联网、tiny smoke需扩大范围、共享NPU状态恶化或发生禁止行为，立即报告`A2-v024 STOP`。

最终只回复：PASS/STOP、Evidence绝对路径、carrier/container identity、实测runtime inventory、bundle与FL identity、package diff、provider ownership、NPU 12+13 pre/post状态、tiny tensor与Dispatch结果、Confirmed/Unknown/Conflict/Potential Blocker。随后停止，不进入canary或GLM Stage。
