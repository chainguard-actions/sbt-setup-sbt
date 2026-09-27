<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `inputs.sbt-runner-version` and then written directly to $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A malicious caller could inject newlines into the version string to poison subsequent steps' outputs.

Locations:

- `action.yml:14`

### github-env-injection (severity: high)

In the 'Setup PATH' step, `$PWD/sbt/bin` is written to $GITHUB_PATH without sanitization (`echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). $PWD is set by `cd "$SBT_TOOLPATH"` where SBT_TOOLPATH is sourced from `steps.cache-paths.outputs.sbt_toolpath`, which was constructed from the untrusted `inputs.sbt-runner-version`. A malicious version string containing newlines could inject arbitrary entries into $GITHUB_PATH.

Locations:

- `action.yml:228`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. All occurrences of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` writes (sbt_toolpath and sbt_cachekey) were replaced with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection from the untrusted `inputs.sbt-runner-version` input.

2. 'Setup PATH' step: Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent) before writing to `$GITHUB_PATH`, replacing the direct `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` with the sanitized variable. This prevents newline injection through the path derived from the untrusted version string.

