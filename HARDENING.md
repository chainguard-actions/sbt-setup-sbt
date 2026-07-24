<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The CI workflow uses action references pinned to mutable version tags instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v7` (line 24) and `actions/setup-java@v5` (line 26). These should be pinned to full SHA digests, e.g. `actions/checkout@<40-char-sha> # v7`.

Locations:

- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:26`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the `test` job also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository default (typically `write-all` for private repos or `read-all` for public repos), granting more access than necessary. A minimal `permissions:` block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is sourced from `inputs.sbt-runner-version` (attacker-controlled) and written directly to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). A newline injected into the input value could allow an attacker to inject arbitrary key-value pairs into `$GITHUB_OUTPUT`, potentially poisoning subsequent steps. Example failing lines: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

1. Pinned actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 and actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 # v5 in .github/workflows/ci.yml. 2. Added top-level `permissions: contents: read` to .github/workflows/ci.yml. 3. Fixed github-env-injection in action.yml by sanitizing the attacker-controlled `SBT_RUNNER_VERSION` input with `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` before writing it to $GITHUB_OUTPUT, replacing all uses of the raw variable in output writes with the sanitized `SAFE_SBT_RUNNER_VERSION`.

