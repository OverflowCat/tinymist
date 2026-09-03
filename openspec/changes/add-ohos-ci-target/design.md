## Context

The main CI workflow already owns Rust checks and invokes the generated cargo-dist release workflow. OpenHarmony requires both a Rust standard library target and an external SDK linker, while cargo-dist's target matrix in this repository does not model OHOS.

## Goals / Non-Goals

**Goals:**

- Make OpenHarmony compilation a visible, independently diagnosable CI job.
- Exercise the same `tinymist` CLI binary users would run on the target.
- Pin the external setup action and SDK inputs so failures can be reproduced.

**Non-Goals:**

- Produce or publish an OpenHarmony release artifact.
- Run the cross-compiled binary on an emulator or device.
- Add HarmonyOS application packaging or editor-extension packaging.

## Decisions

### Add a dedicated job to the main CI workflow

The OHOS check will live in `.github/workflows/ci.yml` as a separate Ubuntu job. Keeping it outside the cargo-dist-generated release workflow avoids hand-extending a generated target matrix and keeps release behavior unchanged. Adding OHOS to an existing host check was rejected because SDK setup and cross-compilation failures would be harder to isolate.

### Target 64-bit ARM OpenHarmony

The job will compile `tinymist-cli` for `aarch64-unknown-linux-ohos`, the primary device architecture and a Rust target with distributed standard-library artifacts. Additional OHOS architectures are deferred until there is a concrete distribution or testing need.

### Use the SDK native component and explicit linker configuration

The setup action will install only the native SDK component. The job will export `CARGO_TARGET_AARCH64_UNKNOWN_LINUX_OHOS_LINKER` using the action's `ohos_sdk_native` output before invoking Cargo. This is sufficient for Rust and build-script linker discovery while avoiding unrelated SDK downloads.

### Compile with the lockfile and workspace warning policy

The job will run `cargo build --locked -p tinymist-cli --bin tinymist --target aarch64-unknown-linux-ohos`. A build, rather than `cargo check`, verifies final target linking against the OpenHarmony SDK. The workflow-level `RUSTFLAGS=-Dwarnings` remains in effect.

## Risks / Trade-offs

- [Some transitive native dependencies may not support OHOS] -> Use GitHub Actions trial results to identify the narrowest source-level or feature adjustment required; do not weaken the target build to a host-only check.
- [SDK downloads increase CI time and storage] -> Install only the native component and retain the setup action's cache.
- [A floating action tag could change behavior] -> Pin the setup action to its released commit SHA and annotate the corresponding version.
