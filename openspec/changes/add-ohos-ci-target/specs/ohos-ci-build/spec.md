## Purpose

Ensure Tinymist remains compilable for a supported OpenHarmony target by exercising the target toolchain in continuous integration.

## ADDED Requirements

### Requirement: OpenHarmony target compilation
The CI system SHALL compile the Tinymist command-line binary for `aarch64-unknown-linux-ohos` with the repository lockfile and warnings denied.

#### Scenario: Normal CI run
- **WHEN** the main CI workflow runs for a configured push, pull request, or manual dispatch
- **THEN** an OpenHarmony job installs the required SDK and Rust target and compiles the Tinymist command-line binary

#### Scenario: Target-specific regression
- **WHEN** Tinymist or one of its locked dependencies does not compile or link for the OpenHarmony target
- **THEN** the OpenHarmony CI job fails

### Requirement: Reproducible OpenHarmony environment
The CI system MUST select explicit OpenHarmony SDK and setup-action versions and use the SDK's target linker.

#### Scenario: SDK provisioning
- **WHEN** the OpenHarmony job configures its build environment
- **THEN** it uses the selected SDK's native toolchain for the target compiler and linker

### Requirement: OpenHarmony binary artifact
The CI system SHALL archive the compiled OpenHarmony Tinymist binary in the same target-keyed layout consumed by downstream editor packaging jobs.

#### Scenario: Successful OpenHarmony build
- **WHEN** the OpenHarmony target build succeeds
- **THEN** the workflow uploads an `aarch64-unknown-linux-ohos` binary artifact that downstream jobs can download by target name

### Requirement: Signed OpenHarmony binary
The CI system SHALL sign the OpenHarmony Tinymist binary before creating any downstream binary or VSIX artifact.

#### Scenario: CI signing
- **WHEN** the release-mode OpenHarmony binary has been compiled
- **THEN** the workflow invokes the OpenHarmony SDK's `binary-sign-tool` dependency in self-sign mode before packaging it

#### Scenario: Signing failure
- **WHEN** the signing tool cannot sign the compiled binary
- **THEN** the OpenHarmony job fails before uploading an unsigned artifact

### Requirement: Experimental OpenHarmony VSIX artifacts
The CI system SHALL include OpenHarmony ARM64 in the VS Code packaging matrix and produce downloadable Tinymist and Typst Preview VSIX artifacts containing the OpenHarmony binary.

#### Scenario: Normal branch CI run
- **WHEN** the OpenHarmony binary and shared VS Code assets are available
- **THEN** the editor packaging matrix produces uniquely named OHOS ARM64 VSIX artifacts

#### Scenario: VSIX target compatibility
- **WHEN** the VSIX tool rejects the native `ohos-arm64` platform identifier
- **THEN** the workflow uses `linux-arm64` for package metadata while retaining unique `ohos-arm64` filenames and artifact names

#### Scenario: GPU viewer unavailable
- **WHEN** the OpenHarmony matrix entry packages editor extensions without an OpenHarmony GPU viewer binary
- **THEN** GPU viewer download, packaging, and upload steps are skipped for that entry

#### Scenario: Marketplace release
- **WHEN** a tagged release or nightly publish workflow runs
- **THEN** the experimental OHOS VSIX is excluded from VS Code Marketplace and Open VSX publishing until a supported platform identifier is established
