<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses action references pinned to mutable tags instead of full 40-character commit SHAs. `actions/checkout@v7` and `actions/setup-java@v5` can be silently updated by the action authors, enabling supply-chain attacks. They must be pinned to a full SHA (e.g., `actions/checkout@<40-hex-sha> # v7`).

Locations:

- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:26`

### permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and the single `test` job also has no `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may include `contents: write` and other broad scopes). A minimal `permissions:` block should be added.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the env var `SBT_RUNNER_VERSION` is populated from `${{ inputs.sbt-runner-version }}` (caller-controlled) and then written directly to `$GITHUB_OUTPUT` in multiple echo statements (e.g., `echo "sbt_toolpath=...\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker supplying a newline-containing version string could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially hijacking subsequent step outputs.

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, github-env-injection

**Notes:**

1. ci.yml: Pinned actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7 and actions/setup-java@v5 → @03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 # v5. 2. ci.yml: Added top-level `permissions: contents: read` block (minimum needed for checkout-based CI). 3. action.yml: In the 'Set up cache paths' step, added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` to strip newlines from the caller-controlled sbt-runner-version input before writing it to $GITHUB_OUTPUT, preventing newline injection attacks.

