## 1. OpenHarmony CI Build

- [x] 1.1 Add a dedicated OHOS job with pinned SDK setup, Rust target installation, and explicit linker configuration; verify the workflow parses locally.
- [x] 1.2 Build `tinymist` for `aarch64-unknown-linux-ohos` with the lockfile on the fork and inspect the GitHub Actions result.
- [x] 1.3 Confirm no target-specific source workarounds are required and verify the OHOS GitHub Actions job succeeds without weakening warning or lockfile checks.
- [x] 1.4 Sign the OHOS binary with the pinned `ohos-binary-sign` package dependency under QEMU before packaging and verify the signing step succeeds in GitHub Actions.

## 2. Validation

- [x] 2.1 Validate the OpenSpec change strictly and inspect the final workflow diff.

## 3. OpenHarmony Artifacts

- [x] 3.1 Produce and upload a release-mode OHOS binary archive in the downstream packaging layout, then verify its artifact is available in GitHub Actions.
- [x] 3.2 Add OHOS ARM64 to the VS Code packaging matrix, skip unsupported GPU viewer steps, and verify the workflow parses locally.
- [x] 3.3 Trial `ohos-arm64` as the VSIX target in GitHub Actions and record whether `vsce` accepts it.
- [x] 3.4 Resolve any VSIX target incompatibility without publishing a misleading Marketplace package, then verify downloadable signed OHOS Tinymist and Typst Preview VSIX artifacts are produced.

## 4. Final Validation

- [x] 4.1 Strictly validate the expanded OpenSpec change and inspect the final workflow diff.
