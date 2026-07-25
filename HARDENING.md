<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml references two actions using mutable tag-based refs instead of pinned 40-character commit SHAs. `actions/checkout@v7` and `actions/setup-java@v5` can be silently updated to point to different (potentially malicious) code. They must be pinned to full SHA digests (e.g., `actions/checkout@<40-hex-sha> # v7`).

Locations:

- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:24`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and the `test` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository default (which may be `write-all` for private repos or `read-all` for public repos). A minimal explicit `permissions:` block (e.g., `contents: read`) should be added.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In action.yml's 'Set up cache paths' step, the env var `$SBT_RUNNER_VERSION` (populated from `inputs.sbt-runner-version` via `SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`) is written directly to `$GITHUB_OUTPUT` multiple times without the required newline-stripping sanitization (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a version string containing a newline character could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps. Affected lines include: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

1. Pinned actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 and actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 # v5 in .github/workflows/ci.yml. 2. Added top-level `permissions: contents: read` to .github/workflows/ci.yml. 3. Fixed github-env-injection in action.yml by computing SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') and using that sanitized variable in all GITHUB_OUTPUT writes that previously used $SBT_RUNNER_VERSION directly (sbt_toolpath for Windows/macOS/Linux branches and sbt_cachekey).

