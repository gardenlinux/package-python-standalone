# package-python-standalone

Builds a compliance-clean [CPython standalone](https://github.com/astral-sh/python-build-standalone) distribution for use in the [Garden Linux](https://github.com/gardenlinux/gardenlinux) test framework.

## Why this exists

The Garden Linux test framework embeds a self-contained Python runtime (via `tests/util/build_runtime.sh`) sourced from [astral-sh/python-build-standalone](https://github.com/astral-sh/python-build-standalone). On Linux, the upstream build compiles CPython with `--with-dbmliborder=bdb`, which statically links **Oracle Berkeley DB (libdb-6.0)** into `_dbm.cpython-*.so`.

Oracle Berkeley DB is dual-licensed under AGPL and a commercial license. The commercial license terms are **not permitted at SAP**, and the presence of BDB code in the binary causes failures in SBOM and license compliance scans.

## What this repo does

This repo rebuilds CPython from the same upstream source (`astral-sh/python-build-standalone`) with a single patch applied to `cpython-unix/build-cpython.sh`:

- Sets `ac_cv_have_libdb=no` — disables libdb detection at `./configure` time
- Switches `--with-dbmliborder=gdbm` — uses GNU dbm instead of Berkeley DB

The resulting `_dbm.cpython-*.so` contains **no Oracle BDB code**.

## What stays the same

Release assets use **identical naming** to the upstream astral-sh releases:

```
cpython-3.14.7+20260901-x86_64-unknown-linux-gnu-install_only.tar.gz
cpython-3.14.7+20260901-aarch64-unknown-linux-gnu-install_only.tar.gz
```

This means `build_runtime.sh` and `update_runtime.py` in the Garden Linux test framework require **no changes** — only `PYTHON_REPO_OWNER` and `PYTHON_REPO_NAME` in `tests/util/python.env.sh` are updated to point here instead of `astral-sh/python-build-standalone`.

## Build

Builds run automatically on push, daily schedule, and `workflow_dispatch`. Two parallel jobs run on GitHub-hosted runners:

- `ubuntu-24.04` → `x86_64-unknown-linux-gnu`
- `ubuntu-24.04-arm` → `aarch64-unknown-linux-gnu`

Both jobs clone `astral-sh/python-build-standalone` at the pinned `UPSTREAM_TAG`, apply `patches/no-libdb.patch`, build the Python toolchain using the upstream Docker-based build system, and publish the patched tarballs as a GitHub release.

## Patch

See [`patches/no-libdb.patch`](patches/no-libdb.patch).
