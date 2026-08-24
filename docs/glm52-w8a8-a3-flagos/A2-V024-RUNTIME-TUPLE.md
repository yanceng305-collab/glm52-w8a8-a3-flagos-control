# A2-v024 Measured Runtime Tuple

> Legacy supplemental file introduced by `e9999d25`. Canonical immutable run snapshot：`results/A2-v024/20260824T025250Z.md`。Future runs不得继续在docs root新增或更新runtime result文件。

Timestamp: 2026-08-24T02:52Z
Evidence: /data/tiankuan/zyg/evidence-a2-v024-20260824T025250Z

## Carrier

| Field | Value |
|---|---|
| Image | quay.io/ascend/vllm-ascend@sha256:1c36469fe1cd2335850eb2318bd3562471c34d5fd8f9f2affb0afc745ce39585 |
| Image ID | sha256:220a47883e42efacb201117d21ba95cc9693d539788ad0345d766e6dc9f4d7bf |
| Tag | nightly-releases-v0.24.0rc-a3 |
| Created | 2026-07-31T05:09:35Z |

## Runtime (measured in container preflight)

| Component | Version |
|---|---|
| OS | Ubuntu 22.04.5 LTS |
| Python | 3.12.13 |
| CANN | 9.0.1 (V100R001C10SPC002B220) |
| torch | 2.10.0+cpu |
| torch_npu | 2.10.0.post2 |
| vllm | 0.24.0+empty |
| triton (pip) | 3.5.0 |
| triton (runtime, from triton-ascend overlay) | 3.2.0 |
| triton-ascend (pre-uninstall) | 3.2.1 |
| transformers | 5.13.0 |

## A2-v024 State

| Gate | Status |
|---|---|
| Carrier digest | PASS |
| FL Git identity | PASS |
| FlagGems v5.3.4 identity | PASS |
| vllm-ascend negative check | PASS |
| FlagTree install | **STOP** (py312 Unresolved) |
| NPU smoke | Not reached |
