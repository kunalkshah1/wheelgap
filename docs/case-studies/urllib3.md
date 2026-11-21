# urllib3: CPython 3.15 and free-threaded 3.14 test dependencies

urllib3 is an HTTP client library for Python. This study checks the development dependencies in its pinned
`uv.lock` with wheelgap **0.13.0** on **21 November 2026**. [urllib3 revision d9f9b0a](https://github.com/urllib3/urllib3/tree/d9f9b0ad5b99247e06aa9a50b7a5747aa5c2e35d)

## Choose targets from the CI matrix

The standard test job runs on Ubuntu, Windows and macOS, including experimental CPython 3.15 and free-threaded
CPython 3.14. This scan models Linux x86_64, Windows x86_64 and Apple silicon macOS. It uses a glibc 2.39
compatibility floor for Linux and macOS 26.0 for the macOS runner. It covers the standard test job's lockfile
selection. [CI matrix](https://github.com/urllib3/urllib3/blob/d9f9b0ad5b99247e06aa9a50b7a5747aa5c2e35d/.github/workflows/ci.yml#L42-L53)

The Nox test session syncs the `dev` group and the `socks`, `brotli`, `zstd` and `h2` extras from the lock.
For the free-threaded job, it leaves out `brotli`. The two scans below mirror those choices.
[Nox test install](https://github.com/urllib3/urllib3/blob/d9f9b0ad5b99247e06aa9a50b7a5747aa5c2e35d/noxfile.py#L17-L54)

## Reproduce the scans

From the pinned checkout, where `uv.lock` and `pyproject.toml` are side by side:

```bash
uvx --from wheelgap==0.13.0 wheelgap uv.lock -p 3.15 \
  -t linux-x86_64,win-amd64,macos-arm64 --glibc 2.39 --macos 26.0 \
  --group dev --extra socks --extra brotli --extra zstd --extra h2 --offline --explain
uvx --from wheelgap==0.13.0 wheelgap uv.lock -p 3.14t \
  -t linux-x86_64,win-amd64,macos-arm64 --glibc 2.39 --macos 26.0 \
  --group dev --extra socks --extra zstd --extra h2 --offline --explain
```

```text
wheelgap 0.13.0 · uv.lock · 64 packages · Python 3.15

skipped 2 non-index packages: hypercorn (git), urllib3 (editable)
✓ All 64 packages have a wheel for every one of 3 targets.
```

The 3.14t run reported wheels for all 63 checked packages, with the same two skipped sources. Both scans returned
exit code 0. The counts are the package rows reported by wheelgap. Installing the test environment also involves
building or preparing the two skipped non-index sources, whose wheel availability is outside these scans.

## Check a compiled dependency with pip

The CPython 3.15 scan includes `brotli==1.2.1`. To check the Windows cell against PyPI:

```bash
python -m pip download --no-deps --only-binary=:all: --index-url https://pypi.org/simple \
  --python-version 3.15 --implementation cp --abi cp315 --platform win_amd64 brotli==1.2.1
```

pip downloaded the three wheels below when given each target's tags. For Linux, pass the
`manylinux_2_N_x86_64` tags from 2.39 down to 2.5 and the three legacy manylinux tags. For macOS, pass
`macosx_26_0_arm64`; pip expands that tag to compatible older floors and universal binaries.

| Target | Wheel downloaded |
| --- | --- |
| Linux x86_64 | `brotli-1.2.1-cp315-cp315-manylinux2014_x86_64.manylinux_2_17_x86_64.whl` |
| Windows x86_64 | `brotli-1.2.1-cp315-cp315-win_amd64.whl` |
| macOS Apple silicon | `brotli-1.2.1-cp315-cp315-macosx_10_15_universal2.whl` |

No wheel changes were indicated for the checked selection. Re-running these scans after a lockfile update checks
whether the new pins still have wheels for the same interpreter and platform combinations.
