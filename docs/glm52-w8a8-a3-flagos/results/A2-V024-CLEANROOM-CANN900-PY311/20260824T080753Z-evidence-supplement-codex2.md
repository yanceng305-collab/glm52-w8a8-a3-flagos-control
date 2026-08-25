# Evidence Supplement Pointer

Task/run: `A2-V024-CLEANROOM-CANN900-PY311` /
`20260824T080753Z`

This is a read-only supplement generated from the closed run. It does not
modify the original Evidence directory, immutable result, container, or Code
repository.

## Server Evidence

- Original Evidence:
  `/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z/`
- Supplemental Evidence:
  `/data/tiankuan/zyg/evidence/A2-V024-CLEANROOM-CANN900-PY311/20260824T080753Z-supplemental-codex2/`
- Acceptance mapping:
  `acceptance-evidence-index.md`
- Original Evidence inventory/checksum:
  `evidence-files.manifest.tsv`, `evidence-files.sha256`,
  `evidence-files.verify.log`
- Retained Work artifact inventory/checksum:
  `work-artifacts.manifest.tsv`, `work-artifacts.sha256`,
  `work-artifacts.verify.log`
- Supplement-authored file checksum:
  `supplemental-files.sha256`, `supplemental-files.verify.log`

## Verification Summary

- Original Evidence inventory: 31 files; checksum verification: successful.
- Retained Work artifact inventory: 11 files; checksum verification:
  successful.
- Supplemental-authored inventory: 8 files; checksum verification:
  successful.
- `evidence-files.manifest.tsv` SHA256:
  `54d9c29edb6bb390201c1561c2e0857406455b691023af939febdf916a203cc2`
- `evidence-files.sha256` SHA256:
  `edee385323c585635ee017fe89644b344bce309d5af809583b76586c25de2365`
- `work-artifacts.manifest.tsv` SHA256:
  `dda89ac2f662a75abda0cbfc19a9db02965bbc403e66f95f31ea9e2409c252b4`
- `work-artifacts.sha256` SHA256:
  `12fa1d723111bd0508ffde4aa859731f2c12077f1f0349b1da238bb3b4711bd0`
- `supplemental-files.sha256` SHA256:
  `6c99a8bff02af93345e8b243d14d6674a991c805710985d945e9429f3879425e`

## Acceptance Status

The complete read-only mapping is in `acceptance-evidence-index.md`. The
following evidence is not recoverable from the existing closed run without
rerunning or adding new instrumentation:

1. A dedicated negative no-copy audit for files, Python, or site-packages from
   another image/container/run.
2. Numeric per-command exit-status records.
3. Direct output tensor device/backend assertions proving no silent CPU
   fallback for the four tiny smoke paths.

These omissions remain explicitly marked as partial/missing. Codex1
Acceptance remains `NEEDS-FOLLOWUP`.

