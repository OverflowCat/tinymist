## Why

Tinymist does not currently compile for OpenHarmony in CI, so target-specific dependency or linker regressions can reach the main branch unnoticed. Add an explicit OHOS build check using the maintained OpenHarmony SDK setup action.

## What Changes

- Add an OpenHarmony CI job for the `aarch64-unknown-linux-ohos` Rust target.
- Install a pinned OpenHarmony SDK with `openharmony-rs/setup-ohos-sdk` before compiling Tinymist.
- Keep OHOS validation separate from release artifact generation and host-platform tests.

## Capabilities

### New Capabilities

- `ohos-ci-build`: Compile Tinymist for the supported OpenHarmony Rust target on every normal CI run.

### Modified Capabilities

None.

## Impact

- `.github/workflows/ci.yml` gains an OHOS-specific build job and SDK dependency.
- GitHub Actions downloads and caches the selected OpenHarmony SDK.
- CI duration and cache storage increase for the new target build.
