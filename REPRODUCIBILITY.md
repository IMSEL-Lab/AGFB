# Reproducibility

This document describes the smallest checks that validate a clean AGFB checkout
and the commands used to regenerate the stored benchmark measurements and
derived tables. The public release is maintained by J.C. Vaught.

## Environment

AGFB uses `uv` with Python 3.11 through 3.13. From the repository root, create
the environment with `uv sync`. The root lockfile installs the five editable
packages and the notebook dependencies. Linux CUDA wheels are selected when the
platform marker matches; CPU and MPS installations remain available on macOS.

## Validation

The package tests cover generator intensity contracts and the metrics pipeline.
Run them with the following command.

```bash
uv run pytest agfb-generators/tests agfb-metrics/tests
```

The public source tree can also be checked with the configured formatter, linter,
and package builder.

```bash
uv run ruff format --check .
uv run ruff check .
uv build
```

`ty` remains available for exploratory type checking, but it is not a passing
release gate yet. The current tree reports known diagnostics in the optional
Triton backend, the real-image path, and notebook analysis code; those findings
are separate from the runtime tests above.

The CPU smoke run exercises the integration path without requiring the full
production catalog or a GPU.

```bash
uv run agfb-bench run --study clean_accuracy --image-size 96 \
    --limit-cells 4 --filter-profile headline --limit-filters 2 \
    --out-dir runs/smoke
```

## Stored measurements

The Parquet shards under `runs/` are the measurements used by the reviewer
notebook and the analysis scripts. They are intentionally retained because they
allow headline values to be recomputed without rerunning the largest GPU jobs.
The analysis scripts write derived CSV files to `runs/_analysis/generated/` by
default, keeping the public checkout independent of any neighboring manuscript
repository.

## Real-image studies

The real-image benchmark supports BSDS500, DRIVE, and BBBC039. These datasets are
not included in this repository. After obtaining them under their own terms,
pass a directory containing the expected dataset folders with `--data-root`.

```bash
uv run agfb-bench run --study edges --data-root /path/to/datasets \
    --datasets bsds500,drive,bbbc039 --out-dir runs/realimg/edges
```

The real-image code reads the source images and annotations, computes boundary
metrics, and stores only the resulting measurements. Dataset licenses and access
conditions remain the responsibility of the dataset providers and the user.

One provenance item remains for maintainer confirmation before final merge. The
real-image module describes its tolerant matching core as adapted from the
`edgecritic` evaluation harness, but this checkout does not include that
upstream project's license or attribution record.
