<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.8** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `${{ inputs.sbt-runner-version }}` (a caller-controlled input) and then written directly to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious caller could supply a version string containing newlines to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially poisoning subsequent steps. Affected lines include the `sbt_toolpath` echo (Windows branch ~line 27, macOS ~line 31, Linux ~line 36) and the `sbt_cachekey` echo (~line 40). The fix is to sanitize the value before writing: `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and use `$safe` in the echo statements.

Locations:

- `action.yml:19`
- `action.yml:27`
- `action.yml:31`
- `action.yml:36`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of the caller-controlled `sbt-runner-version` input before writing to $GITHUB_OUTPUT. At the top of the 'Set up cache paths' run block, added `safe_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` to strip embedded newlines/carriage returns. Replaced all four uses of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT echo statements (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) with `$safe_version`.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 28, 33, 38): Added sanitization for inherited env vars RUNNER_TOOL_CACHE, RUNNER_TEMP, LOCALAPPDATA, and HOME using `printf '%s' "$VAR" | tr -d '\n\r'` before writing their values to $GITHUB_OUTPUT.

2. 'Setup PATH' step (lines 214, 216): Added sanitization for $PWD (derived from the untrusted SBT_TOOLPATH step output) using `safe_pwd=$(printf '%s' "$PWD" | tr -d '\n\r')` before writing to $GITHUB_PATH.

