# Stage A1 Task Contract — A3 Read-only Environment Inventory

状态：Ready

Task ID：`A3-CP-A1`
Parent：`Stage A — Clean Provenance`
执行对象：第一台Ascend A3/910C服务器
执行者：DeepSeek
验收者：Codex
第二台服务器：不需要

## 目标

第一次接触A3服务器只冻结现场事实，为后续`R0-clean` exact tuple设计提供输入。不得安装、卸载、启动模型、创建容器、修改系统或自行决定环境版本。

## 唯一允许的写入

本任务原则上只读。为满足证据留存要求，唯一允许写入是新建一个此前不存在的专用证据目录：

```text
$HOME/a3-readonly-inventory-<hostname>-<UTC timestamp>/
```

目录内只允许保存本任务的raw logs、`commands.tsv`、`REPORT.md`和`SHA256SUMS`。不得覆盖、删除或修改目录外任何文件；若`$HOME`不可写，改用新建的`/tmp/a3-readonly-inventory-...`并在报告标明。

## 允许采集范围

- OS、kernel、architecture、hostname；
- CPU与系统内存基本信息；
- `npu-smi info`、NPU数量/型号、每device HBM、logical presentation、占用进程；
- 现有Python环境安全可导入时的`torch.npu.device_count()`及只读device properties；
- Driver、Firmware、CANN版本与安装路径；
- Python、torch、torch-npu、Docker版本；
- 本机已有Docker images和containers清单；
- 当前pip/distribution/module中是否存在vLLM、vllm-ascend、vllm-plugin-FL、FlagGems、FlagCX、FlagTree、Triton/triton-ascend；
- 当前PATH/PYTHONPATH/LD_LIBRARY_PATH及显式Ascend/HCCL/FlagOS相关变量；
- HCCL/NPU网络基础信息；
- 磁盘空间；
- 指定常见模型根目录中是否存在GLM-5.2-W8A8或Qwen canary权重。

## 明确禁止

- `pip/uv/conda install|uninstall`；
- `apt/yum/dnf/rpm`安装、卸载或修改；
- `docker pull/build/run/create/rm/rmi`；
- 创建或启动container；
- 修改或持久化环境变量；不得`source`新的CANN/Driver环境脚本后再把结果冒充当前状态；
- 修改Driver、Firmware、CANN、网络、路由、HCCL配置；
- 删除/移动/覆盖文件；
- `kill/pkill/killall`或改变任何进程；
- clone、commit、push或修改Git仓库；
- 启动Qwen、GLM、vLLM或任何模型服务；
- 使用`sudo`绕过读取权限；权限不足记为Unknown；
- 下载image、wheel、模型、源码或外部资料；
- 自行选择R0-clean版本组合、搭建环境或进入下一Stage。

## 原始证据要求

每条命令必须保存：完整命令、UTC时间、stdout、stderr和exit code。失败命令不能隐藏；以独立raw log保留。报告中的每个Confirmed结论必须链接到raw log文件。

## 结构化报告

`REPORT.md`必须包含：

1. Executive summary；
2. Inventory表：字段、观测值、状态、raw evidence；
3. `Confirmed`；
4. `Unknown`；
5. `Conflict`；
6. `Potential blocker`；
7. 五个特别问题的直接回答；
8. 命令失败/权限不足；
9. 未执行事项声明。

特别问题：

- 真实A3向PyTorch暴露多少NPU device？
- 官方物理规格`8×128GB`与现场logical presentation是什么关系？
- 当前Driver/CANN是否有足够证据支撑拟议clean FlagOS stack；证据不足必须写Unknown/Potential blocker，不得自行冻结tuple；
- Host/current Python/Docker image中是否存在vllm-ascend痕迹；存在只表示现场事实，不自动判定未来R0-clean违规；
- 本机是否已经存在合适的neutral CANN/torch-npu base image；没有则只记录，不下载。

## DeepSeek完整执行提示词

```text
你正在执行项目任务A3-CP-A1：A3 Read-only Environment Inventory。

这是第一次接触第一台Ascend A3/910C服务器。任务唯一目标是采集现有事实并保存证据。不要安装、卸载、启动模型、创建容器、修改系统或选择R0-clean版本。

【控制边界】
1. 只执行读取/查询命令。
2. 唯一允许写入：新建一个专用证据目录，并在其中保存raw logs、commands.tsv、REPORT.md、SHA256SUMS。
3. 不得使用sudo绕过权限；Permission denied记为Unknown。
4. 不得source任何新的CANN/Driver set_env.sh；必须记录登录shell的真实当前环境。
5. 不得执行pip/uv/conda install或uninstall；不得apt/yum/dnf修改；不得docker pull/build/run/create/rm/rmi；不得修改网络、Driver、Firmware、CANN；不得kill进程；不得clone/push代码；不得启动模型。
6. 不得访问第二台服务器。
7. 不得自行决定R0-clean版本tuple或开始环境搭建。
8. 不得dump完整env、进程完整argv、Docker credential/proxy配置或任何token/password；只采集本提示词列出的限定字段。

【证据目录】
在当前用户HOME下创建全新目录：

  OUT="$HOME/a3-readonly-inventory-$(hostname)-$(date -u +%Y%m%dT%H%M%SZ)"
  umask 077
  mkdir -p "$OUT/raw"

如果HOME不可写，改用/tmp下的全新同名目录，并在REPORT.md说明。不得覆盖已有目录。

在shell中定义以下只用于记录的函数：

  run() {
    label="$1"; shift
    logfile="$OUT/raw/${label}.log"
    start="$(date -u +%Y-%m-%dT%H:%M:%SZ)"
    {
      echo "__START_UTC__=$start"
      printf '__COMMAND__='; printf ' %q' "$@"; echo
      "$@"
      rc=$?
      echo "__EXIT_CODE__=$rc"
      echo "__END_UTC__=$(date -u +%Y-%m-%dT%H:%M:%SZ)"
    } >"$logfile" 2>&1
    rc=$(grep '^__EXIT_CODE__=' "$logfile" | tail -1 | cut -d= -f2)
    printf '%s\t%s\t%s\n' "$label" "${rc:-unknown}" "$logfile" >>"$OUT/commands.tsv"
    return 0
  }

  run_sh() {
    label="$1"; shift
    cmd="$*"
    logfile="$OUT/raw/${label}.log"
    {
      echo "__START_UTC__=$(date -u +%Y-%m-%dT%H:%M:%SZ)"
      echo "__SHELL_COMMAND__=$cmd"
      bash -o pipefail -c "$cmd"
      rc=$?
      echo "__EXIT_CODE__=$rc"
      echo "__END_UTC__=$(date -u +%Y-%m-%dT%H:%M:%SZ)"
    } >"$logfile" 2>&1
    rc=$(grep '^__EXIT_CODE__=' "$logfile" | tail -1 | cut -d= -f2)
    printf '%s\t%s\t%s\n' "$label" "${rc:-unknown}" "$logfile" >>"$OUT/commands.tsv"
    return 0
  }

先写入任务元数据：

  printf 'task_id\tA3-CP-A1\nstarted_utc\t%s\nhostname\t%s\nuser\t%s\npwd\t%s\n' \
    "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "$(hostname)" "$(id -un)" "$PWD" \
    >"$OUT/task-metadata.tsv"

【必须执行的只读采集】

基础系统：

  run 001-uname uname -a
  run 002-os-release cat /etc/os-release
  run 003-hostname hostname
  run_sh 004-hostnamectl 'command -v hostnamectl >/dev/null && timeout 15s hostnamectl || true'
  run 005-architecture uname -m
  run 006-long-bit getconf LONG_BIT
  run 007-id id
  run 008-lscpu lscpu
  run 009-free free -h
  run 010-meminfo cat /proc/meminfo
  run_sh 011-numa 'command -v numactl >/dev/null && numactl --hardware || true'
  run 012-mounts cat /proc/mounts
  run 013-disk df -hT
  run 014-inodes df -ih

NPU与进程：

  run_sh 020-npu-smi-path 'command -v npu-smi || true; ls -l /usr/local/bin/npu-smi 2>/dev/null || true'
  run_sh 021-npu-smi-help 'command -v npu-smi >/dev/null && npu-smi -h || true'
  run_sh 022-npu-smi-info 'command -v npu-smi >/dev/null && npu-smi info || true'
  run_sh 023-npu-devices 'find /dev -maxdepth 1 \( -name "davinci*" -o -name "devmm_svm" -o -name "hisi_hdc" \) -printf "%f\t%y\t%M\t%u:%g\n" 2>/dev/null | sort'
  run_sh 024-ascend-pci 'command -v lspci >/dev/null && lspci -nn | grep -Ei "Huawei|Ascend|Davinci" || true'
  run_sh 025-npu-processes 'ps -eo pid,ppid,user,lstart,etime,pcpu,pmem,comm --sort=pid | grep -Ei "vllm|torchrun|ray|mindie|python|ascend|npu" | grep -v grep || true'

Driver、Firmware、CANN：

  run_sh 030-ascend-root 'ls -la /usr/local/Ascend 2>/dev/null || true; find /usr/local/Ascend -maxdepth 3 -type l -printf "%p -> %l\n" 2>/dev/null | sort'
  run_sh 031-driver-files 'for f in /usr/local/Ascend/driver/version.info /usr/local/Ascend/driver/version.cfg /etc/ascend_install.info; do echo "===== $f ====="; [ -r "$f" ] && cat "$f" || echo UNREADABLE_OR_ABSENT; done'
  run_sh 032-cann-paths 'for p in /usr/local/Ascend/ascend-toolkit/latest /usr/local/Ascend/nnal/atb/latest /usr/local/Ascend/latest; do echo "===== $p ====="; [ -e "$p" ] && readlink -f "$p" || echo ABSENT; done'
  run_sh 033-version-files 'find /usr/local/Ascend -maxdepth 5 -type f \( -name "version.info" -o -name "version.cfg" -o -name "ascend_toolkit_install.info" \) -print 2>/dev/null | sort | while read -r f; do echo "===== $f ====="; sed -n "1,160p" "$f" 2>&1; done'
  run_sh 034-packages-deb-rpm 'if command -v dpkg-query >/dev/null; then dpkg-query -W 2>/dev/null | grep -Ei "ascend|cann|torch|hccl|atb" || true; fi; if command -v rpm >/dev/null; then rpm -qa | grep -Ei "ascend|cann|torch|hccl|atb" || true; fi'

Python与现有packages：

  run_sh 040-python-paths 'command -v python3 || true; command -v python || true; command -v pip3 || true; command -v pip || true'
  run_sh 041-python-version 'python3 --version 2>&1 || python --version 2>&1 || true'
  run_sh 042-pip-version 'python3 -m pip --version 2>&1 || true'
  run_sh 043-pip-list 'python3 -m pip list --format=freeze 2>&1 || true'

使用下面heredoc在证据目录创建`$OUT/package_inventory.py`，然后通过`run`记录执行。该脚本只读取distribution metadata和module spec，不安装或修改package：

cat >"$OUT/package_inventory.py" <<'PY'
from importlib import metadata, util
import json, platform, sys
targets = {
    "vllm", "vllm-ascend", "vllm-plugin-fl", "flag-gems", "flag_gems",
    "flagcx", "flagtree", "triton", "triton-ascend", "torch", "torch-npu"
}
def norm(s): return s.lower().replace("_", "-").replace(".", "-")
rows = []
for d in metadata.distributions():
    name = d.metadata.get("Name", "")
    if norm(name) in {norm(x) for x in targets}:
        rows.append({"name": name, "version": d.version, "root": str(d.locate_file(""))})
modules = {}
for name in ["vllm", "vllm_ascend", "vllm_fl", "flag_gems", "flagcx", "flagtree", "triton", "torch", "torch_npu"]:
    try:
        spec = util.find_spec(name)
        modules[name] = None if spec is None else {"origin": spec.origin, "locations": list(spec.submodule_search_locations or [])}
    except Exception as e:
        modules[name] = {"error": repr(e)}
print(json.dumps({"python": sys.version, "executable": sys.executable, "platform": platform.platform(), "distributions": sorted(rows, key=lambda x: x["name"].lower()), "modules": modules}, indent=2, ensure_ascii=False))
PY
run 044-package-inventory env PYTHONDONTWRITEBYTECODE=1 python3 "$OUT/package_inventory.py"

如果且仅如果044结果表明torch与torch_npu均已存在，并且普通import不会触发安装、编译或写入，再创建并运行下面只读probe；不得set_device、分配tensor或执行kernel。若import出现写入/编译迹象立即停止probe并记Unknown。否则创建raw/045-torch-npu-probe.log并写明SKIPPED及原因：

cat >"$OUT/torch_npu_probe.py" <<'PY'
import json
import torch
import torch_npu
out = {
    "torch": getattr(torch, "__version__", "unknown"),
    "torch_npu": getattr(torch_npu, "__version__", "unknown"),
    "npu_is_available": bool(torch.npu.is_available()),
    "npu_device_count": int(torch.npu.device_count()),
    "properties": [],
}
for i in range(out["npu_device_count"]):
    try:
        out["properties"].append({"index": i, "repr": repr(torch.npu.get_device_properties(i))})
    except Exception as e:
        out["properties"].append({"index": i, "error": repr(e)})
print(json.dumps(out, indent=2, ensure_ascii=False))
PY
run 045-torch-npu-probe env PYTHONDONTWRITEBYTECODE=1 python3 "$OUT/torch_npu_probe.py"

环境变量：只采集下列显式变量，不得dump完整env或任何credential变量。

  run_sh 050-selected-env 'for v in PATH PYTHONPATH LD_LIBRARY_PATH ASCEND_HOME_PATH ASCEND_OPP_PATH ASCEND_AICPU_PATH TOOLCHAIN_HOME HCCL_IF_IP HCCL_SOCKET_IFNAME HCCL_BUFFSIZE SOC_VERSION ASCEND_RT_VISIBLE_DEVICES NPU_VISIBLE_DEVICES VLLM_PLUGINS VLLM_FL_PLATFORM VLLM_VENDOR TRITON_ALL_BLOCKS_PARALLEL FLAGCX_PATH; do eval "val=\${$v-}"; printf "%s=%s\n" "$v" "$val"; done'

Docker现状（只列出，不pull/run/inspect外部registry）：

  run_sh 060-docker-path-version 'command -v docker || true; docker version 2>&1 || true'
  run_sh 061-docker-info 'docker info --format "ServerVersion={{.ServerVersion}}\nStorageDriver={{.Driver}}\nOSType={{.OSType}}\nOperatingSystem={{.OperatingSystem}}\nArchitecture={{.Architecture}}\nKernelVersion={{.KernelVersion}}\nDockerRootDir={{.DockerRootDir}}\nCgroupDriver={{.CgroupDriver}}\nCgroupVersion={{.CgroupVersion}}\nNCPU={{.NCPU}}\nMemTotal={{.MemTotal}}" 2>&1 || true'
  run_sh 062-docker-images 'docker images --digests --no-trunc 2>&1 || true'
  run_sh 063-docker-containers 'docker ps -a --no-trunc --format "{{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Names}}\t{{.Networks}}" 2>&1 || true'

网络与HCCL基础信息：

  run_sh 070-ip-address 'command -v ip >/dev/null && ip -brief address || true'
  run_sh 071-ip-route 'command -v ip >/dev/null && ip route show table all || true'
  run_sh 072-ip-link 'command -v ip >/dev/null && ip -details link show || true'
  run_sh 073-hccn-tool 'command -v hccn_tool || true; ls -l /usr/local/Ascend/driver/tools/hccn_tool 2>/dev/null || true'
  run_sh 074-hccn-ip 'tool=$(command -v hccn_tool 2>/dev/null || true); [ -z "$tool" ] && tool=/usr/local/Ascend/driver/tools/hccn_tool; if [ -x "$tool" ]; then for dev in /dev/davinci[0-9]*; do [ -e "$dev" ] || continue; i=${dev##*davinci}; echo "===== device $i ====="; "$tool" -i "$i" -ip -g 2>&1; done; else echo HCCN_TOOL_ABSENT; fi'
  run_sh 075-hccn-config 'for f in /etc/hccn.conf /etc/ascend_hccn.conf; do echo "===== $f ====="; [ -r "$f" ] && cat "$f" || echo UNREADABLE_OR_ABSENT; done'

模型与磁盘：不要遍历整个根文件系统，不要读取大权重内容，不要计算权重hash。只检查常见根目录，限制深度和时间：

  run_sh 080-model-roots 'for d in /data /models /model /mnt/models "$HOME/.cache/modelscope/hub" "$HOME/.cache/huggingface/hub" /root/.cache/modelscope/hub /root/.cache/huggingface/hub; do if [ -d "$d" ]; then printf "PRESENT\t%s\n" "$d"; timeout 30s du -sh "$d" 2>&1 || true; else printf "ABSENT\t%s\n" "$d"; fi; done'
  run_sh 081-model-candidates 'for d in /data /models /model /mnt/models "$HOME/.cache/modelscope/hub" "$HOME/.cache/huggingface/hub" /root/.cache/modelscope/hub /root/.cache/huggingface/hub; do [ -d "$d" ] || continue; echo "===== $d ====="; timeout 60s find "$d" -maxdepth 5 -type d \( -iname "*GLM-5.2*" -o -iname "*GLM_5_2*" -o -iname "*Qwen3.6-27B*" -o -iname "*Qwen3-4B*" \) -print 2>&1 || true; done'
  run_sh 082-model-metadata 'for d in /data /models /model /mnt/models "$HOME/.cache/modelscope/hub" "$HOME/.cache/huggingface/hub" /root/.cache/modelscope/hub /root/.cache/huggingface/hub; do [ -d "$d" ] || continue; timeout 60s find "$d" -maxdepth 7 -type f \( -name "config.json" -o -name "quant_model_description.json" -o -name "model.safetensors.index.json" \) -printf "%p\t%s bytes\n" 2>&1 | grep -Ei "glm.?5.?2|qwen3.?6|qwen3-4b" || true; done'

完成命令后：

1. 生成$OUT/REPORT.md，严格分为Confirmed、Unknown、Conflict、Potential blocker。
2. 对五个特别问题逐项直接回答；没有证据就写Unknown，禁止猜测版本兼容。
3. 明确区分：host/current environment存在vllm-ascend痕迹，不等于未来R0-clean违规；R0-clean仍必须从零且从未安装vllm-ascend。
4. 对Docker images只判断“本机是否已有candidate neutral image”，不要下载；不要仅凭名称宣布其合规，缺少package inventory时写Unknown。
5. 不提出或执行R0-clean exact tuple。
6. 生成校验清单：

     (cd "$OUT" && find . -type f ! -name SHA256SUMS -print0 | sort -z | xargs -0 sha256sum > SHA256SUMS)

7. 最终只返回：任务状态、OUT绝对路径、REPORT.md完整内容、raw log清单、失败/权限不足、Potential blocker、需要Codex决策的事项。
8. 完成后停止，不进入环境搭建或任何下一Stage。
```

## Codex验收与下一步

DeepSeek结果返回后，Codex负责：

1. 核对raw logs与REPORT结论；
2. 更新Confirmed/Unknown/Conflict/Potential blocker；
3. 重新判断Driver/Firmware/CANN/Python/torch/torch-npu/compiler profile；
4. 决定R0-clean exact tuple；
5. 只有用户批准后，生成下一条Clean Provenance环境构建任务。

DeepSeek不得自行执行第3～5项。
