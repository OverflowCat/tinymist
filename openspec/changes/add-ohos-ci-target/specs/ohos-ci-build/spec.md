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
