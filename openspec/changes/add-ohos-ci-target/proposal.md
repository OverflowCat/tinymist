## Why

Tinymist does not currently compile for OpenHarmony in CI, so target-specific dependency or linker regressions can reach the main branch unnoticed. Add an explicit OHOS build check using the maintained OpenHarmony SDK setup action.

## What Changes

- Add an OpenHarmony CI job for the `aarch64-unknown-linux-ohos` Rust target.
- Install a pinned OpenHarmony SDK with `openharmony-rs/setup-ohos-sdk` before compiling Tinymist.
- Sign the compiled OHOS binary with the pinned `ohos-binary-sign` Node.js package before packaging it.
- Upload the compiled OHOS command-line binary in the archive layout consumed by editor packaging jobs.
- Add an OHOS entry to the VS Code packaging matrix and produce downloadable Tinymist and Typst Preview VSIX artifacts for trial use.
- Keep experimental OHOS VSIX artifacts out of Marketplace publishing until the editor platform identifier is confirmed.

## Capabilities

### New Capabilities

- `ohos-ci-build`: Compile and package Tinymist for the supported OpenHarmony Rust target on every normal CI run.

### Modified Capabilities

None.

## Impact

- `.github/workflows/ci.yml` gains an OHOS-specific build job, SDK dependency, and binary artifact.
- `.github/workflows/build-vscode-main.yml` gains an experimental OHOS packaging matrix entry.
- GitHub Actions downloads and caches the selected OpenHarmony SDK.
- CI duration, artifact storage, and cache storage increase for the new target build and VSIX packages.
