<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file uses mutable tag-based action references instead of pinned SHA commits. `actions/checkout@v6` and `actions/setup-java@v5` are both tag references that can be silently changed by the upstream maintainer, enabling supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:23`

### missing-permissions (severity: medium)

The workflow file `ci.yml` has no top-level `permissions:` key and the `test` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access to contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the env var `SBT_RUNNER_VERSION` (sourced from `inputs.sbt-runner-version`, a caller-controlled input) is written directly to `$GITHUB_OUTPUT` via `echo` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker supplying a version string containing a newline character could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially poisoning subsequent steps. Affected lines include: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`, `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`, and `echo "sbt_diskcachekey=..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

1. unpinned-uses: Pinned `actions/checkout@v6` to SHA `d23441a48e516b6c34aea4fa41551a30e30af803` and `actions/setup-java@v5` to SHA `03ad4de0992f5dab5e18fcb136590ce7c4a0ac95` in `.github/workflows/ci.yml`, preserving the tag as a comment. 2. missing-permissions: Added `permissions: {}` at the workflow top level and `permissions: contents: read` at the `test` job level (checkout requires read access to contents). 3. github-env-injection: In `action.yml`'s 'Set up cache paths' step, sanitized the caller-controlled `SBT_RUNNER_VERSION` input by computing `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before using it in all `$GITHUB_OUTPUT` writes, preventing newline injection attacks.

