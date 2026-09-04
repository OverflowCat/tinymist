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

The setup action will install the native SDK component. The job will export `CARGO_TARGET_AARCH64_UNKNOWN_LINUX_OHOS_LINKER` using the action's `ohos_sdk_native` output before invoking Cargo.

### Compile a distributable binary with the lockfile and workspace warning policy

The job will run a release build of `tinymist-cli` for `aarch64-unknown-linux-ohos`. A build, rather than `cargo check`, verifies final target linking against the OpenHarmony SDK. The workflow-level `RUSTFLAGS=-Dwarnings` remains in effect. The binary will be archived using the `artifacts-build-local-<rust-target>` name and `tinymist-<rust-target>/tinymist` layout already consumed by the VS Code workflow.

### Sign the binary before distribution

After the release build, the job will install Node.js 24 and the pinned `ohos-binary-sign@1.0.0` npm package, then invoke the package's bundled `platform/openharmony-arm64/binary-sign-tool` dependency with `-selfSign 1`. Because that dependency is an OpenHarmony ARM64 executable, the workflow runs it under `qemu-aarch64` with a minimal root assembled from the SDK sysroot's musl loader and C++ runtime; it does not load the wrapper's host-platform guard. This signs the binary in place before creating the archive, keeping both the standalone OHOS artifact and the VSIX payload signed while avoiding unprovided certificate secrets for the experimental CI-only packages. A missing tool, emulator, runtime library, or signing error fails the job before upload.

### Make the release workflow wait for the OHOS artifact

The top-level `build` reusable-workflow call will depend on `checks-ohos`. This expresses artifact readiness without polling and lets the existing cross-workflow artifact download mechanism consume the OHOS archive.

### Add an experimental OHOS VS Code matrix entry

`build-vscode-main.yml` will add an `ohos-arm64` entry backed by `aarch64-unknown-linux-ohos`. A GitHub Actions trial confirmed that `vsce` rejects `ohos-arm64` because it is not a recognized VS Code target. The matrix will therefore keep the unique OHOS artifact name while using `linux-arm64`, the nearest supported VS Code platform identifier, only for VSIX metadata.

The OHOS entry will package the Tinymist and Typst Preview extensions but skip the GPU viewer, since no OHOS viewer binary exists. It will run for normal branch CI as an experimental pre-release artifact. Tagged workflows may retain the uniquely named files as GitHub release assets, but Marketplace and Open VSX publication will explicitly filter them out to avoid publishing a colliding `linux-arm64` platform package.

## Risks / Trade-offs

- [Some transitive native dependencies may not support OHOS] -> Use GitHub Actions trial results to identify the narrowest source-level or feature adjustment required; do not weaken the target build to a host-only check.
- [SDK downloads increase CI time and storage] -> Install only the native component and retain the setup action's cache.
- [A floating action tag could change behavior] -> Pin the setup action to its released commit SHA and annotate the corresponding version.
- [VS Code tooling rejects `ohos-arm64` as an unknown target] -> Use `linux-arm64` only for VSIX metadata while retaining the `ohos-arm64` artifact label.
- [Using `linux-arm64` metadata could collide with the GNU/Linux package] -> Keep OHOS VSIX artifacts CI-only and uniquely named until the target editor defines a distinct platform identifier.
- [Self-signed binaries may not satisfy a production device trust policy] -> Keep the packages experimental and out of Marketplace/Open VSX publication; replace self-signing with maintainer-provided signing credentials when a production distribution policy is established.
- [The npm wrapper only supports OpenHarmony ARM64 hosts] -> Extract its bundled `binary-sign-tool` dependency and execute it under QEMU with the SDK sysroot on the Ubuntu cross-compilation runner.
- [QEMU or the target sysroot may not provide all runtime behavior required by the signer] -> Build a minimal QEMU root from the SDK's loader and required libraries, keep the emulated invocation isolated to signing, and fail before upload if it cannot run; the target binary itself remains unmodified unless signing succeeds.
