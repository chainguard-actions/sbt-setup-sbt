<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is set from `${{ inputs.sbt-runner-version }}` (user-controlled input) and then written directly to $GITHUB_OUTPUT multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A malicious caller could inject newlines into the version string to poison subsequent steps via GITHUB_OUTPUT.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is set from `${{ steps.cache-paths.outputs.sbt_toolpath }}` (which is derived from user-controlled `inputs.sbt-runner-version`). The step does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization: `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`. Since $PWD is derived from the user-controlled SBT_TOOLPATH, a newline-containing version string could poison $GITHUB_PATH.

Locations:

- `action.yml:209`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized the user-controlled SBT_RUNNER_VERSION input by adding `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all GITHUB_OUTPUT writes to use `$SAFE_SBT_RUNNER_VERSION` instead of `$SBT_RUNNER_VERSION`.
2. 'Setup PATH' step: Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows variant) before writing to GITHUB_PATH, preventing newline injection via the user-controlled SBT_TOOLPATH-derived $PWD value.

