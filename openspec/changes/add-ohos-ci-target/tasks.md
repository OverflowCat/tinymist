## 1. OpenHarmony CI Build

- [x] 1.1 Add a dedicated OHOS job with pinned SDK setup, Rust target installation, and explicit linker configuration; verify the workflow parses locally.
- [ ] 1.2 Build `tinymist` for `aarch64-unknown-linux-ohos` with the lockfile on the fork and inspect the GitHub Actions result.
- [ ] 1.3 Resolve target-specific build failures without weakening warning or lockfile checks, then verify the OHOS GitHub Actions job succeeds.

## 2. Validation

- [ ] 2.1 Validate the OpenSpec change strictly and inspect the final workflow diff.
