## 1. OpenHarmony CI Build

- [x] 1.1 Add a dedicated OHOS job with pinned SDK setup, Rust target installation, and explicit linker configuration; verify the workflow parses locally.
- [x] 1.2 Build `tinymist` for `aarch64-unknown-linux-ohos` with the lockfile on the fork and inspect the GitHub Actions result.
- [x] 1.3 Confirm no target-specific source workarounds are required and verify the OHOS GitHub Actions job succeeds without weakening warning or lockfile checks.

## 2. Validation

- [x] 2.1 Validate the OpenSpec change strictly and inspect the final workflow diff.
