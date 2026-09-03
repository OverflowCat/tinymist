## Context

The main CI workflow already owns Rust checks and invokes the generated cargo-dist release workflow. OpenHarmony requires both a Rust standard library target and an external SDK linker, while cargo-dist's target matrix in this repository does not model OHOS.

## Goals / Non-Goals

**Goals:**

- Make OpenHarmony compilation a visible, independently diagnosable CI job.
- Exercise the same `tinymist` CLI binary users would run on the target.
- Pin the external setup action and SDK inputs so failures can be reproduced.
- Feed the successful OHOS build into the existing VS Code packaging matrix.
- Produce trial VSIX artifacts without exposing an ambiguous platform package to Marketplace users.

**Non-Goals:**

- Publish an OpenHarmony VSIX to VS Code Marketplace or Open VSX.
- Run the cross-compiled binary on an emulator or device.
- Add HarmonyOS application packaging.

## Decisions

### Add a dedicated job to the main CI workflow

The OHOS check will live in `.github/workflows/ci.yml` as a separate Ubuntu job. Keeping it outside the cargo-dist-generated release workflow avoids hand-extending a generated target matrix and keeps release behavior unchanged. Adding OHOS to an existing host check was rejected because SDK setup and cross-compilation failures would be harder to isolate.

### Target 64-bit ARM OpenHarmony

The job will compile `tinymist-cli` for `aarch64-unknown-linux-ohos`, the primary device architecture and a Rust target with distributed standard-library artifacts. Additional OHOS architectures are deferred until there is a concrete distribution or testing need.

### Use the SDK native component and explicit linker configuration

The setup action will install only the native SDK component. The job will export `CARGO_TARGET_AARCH64_UNKNOWN_LINUX_OHOS_LINKER` using the action's `ohos_sdk_native` output before invoking Cargo. This is sufficient for Rust and build-script linker discovery while avoiding unrelated SDK downloads.

### Compile a distributable binary with the lockfile and workspace warning policy

The job will run a release build of `tinymist-cli` for `aarch64-unknown-linux-ohos`. A build, rather than `cargo check`, verifies final target linking against the OpenHarmony SDK. The workflow-level `RUSTFLAGS=-Dwarnings` remains in effect. The binary will be archived using the `artifacts-build-local-<rust-target>` name and `tinymist-<rust-target>/tinymist` layout already consumed by the VS Code workflow.

### Make the release workflow wait for the OHOS artifact

The top-level `build` reusable-workflow call will depend on `checks-ohos`. This expresses artifact readiness without polling and lets the existing cross-workflow artifact download mechanism consume the OHOS archive.

### Add an experimental OHOS VS Code matrix entry

`build-vscode-main.yml` will add an `ohos-arm64` entry backed by `aarch64-unknown-linux-ohos`. The first trial will pass `ohos-arm64` to `vsce` directly. If `vsce` rejects it, the matrix will keep the unique OHOS artifact name while using the nearest supported VS Code platform identifier only for VSIX metadata.

The OHOS entry will package the Tinymist and Typst Preview extensions but skip the GPU viewer, since no OHOS viewer binary exists. It will run for normal branch CI as an experimental pre-release artifact and be disabled for tagged release/nightly publishing to avoid publishing an unsupported or colliding Marketplace platform package.

## Risks / Trade-offs

- [Some transitive native dependencies may not support OHOS] -> Use GitHub Actions trial results to identify the narrowest source-level or feature adjustment required; do not weaken the target build to a host-only check.
- [SDK downloads increase CI time and storage] -> Install only the native component and retain the setup action's cache.
- [A floating action tag could change behavior] -> Pin the setup action to its released commit SHA and annotate the corresponding version.
- [VS Code tooling may reject `ohos-arm64` as an unknown target] -> Trial the native identifier first, then separate the artifact label from VSIX target metadata if a supported compatibility identifier is required.
- [Using `linux-arm64` metadata could collide with the GNU/Linux package] -> Keep OHOS VSIX artifacts CI-only and uniquely named until the target editor defines a distinct platform identifier.
