<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file ci.yml references GitHub Actions using mutable tag-based refs instead of immutable 40-character commit SHAs. `actions/checkout@v6` (line 25) and `actions/setup-java@v5` (line 27) can be silently updated by the action author, enabling supply-chain attacks. These must be pinned to full SHA digests (e.g., `actions/checkout@<40-char-sha> # v6`).

Locations:

- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:27`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and its only job (`test`) also has no job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, etc.). A minimal `permissions:` block (e.g., `contents: read`) should be added at the top level or on every job.

Locations:

- `.github/workflows/ci.yml:1`

### github-env-injection (severity: high)

In action.yml, the 'Set up cache paths' step sets `SBT_RUNNER_VERSION` from the user-controlled input `inputs.sbt-runner-version` via an `env:` block, then writes that value directly into `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a newline-containing version string could inject arbitrary key=value pairs into the step output context. Each write of `$SBT_RUNNER_VERSION` to `$GITHUB_OUTPUT` must be preceded by sanitization.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`
- `action.yml:43`
- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection

**Notes:**

Fixed three findings: (1) Pinned actions/checkout@v6 to SHA df4cb1c069e1874edd31b4311f1884172cec0e10 and actions/setup-java@v5 to SHA 03ad4de0992f5dab5e18fcb136590ce7c4a0ac95 in .github/workflows/ci.yml. (2) Added top-level `permissions: contents: read` block to ci.yml. (3) In action.yml's 'Set up cache paths' step, added sanitization of the user-controlled SBT_RUNNER_VERSION input using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all GITHUB_OUTPUT writes that used $SBT_RUNNER_VERSION with $SAFE_SBT_RUNNER_VERSION to prevent newline injection attacks.

