<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var $SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (an untrusted caller-controlled input) and written to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). A caller could supply a version string containing newline characters to inject arbitrary key=value pairs into $GITHUB_OUTPUT, potentially overwriting subsequent outputs. Affected lines include the sbt_toolpath writes (Windows, macOS, and Linux branches) and the sbt_cachekey write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, $SBT_TOOLPATH (sourced from steps.cache-paths.outputs.sbt_toolpath, which was derived from the caller-controlled input inputs.sbt-runner-version) is used in 'cd "$SBT_TOOLPATH"', and then $PWD is written to $GITHUB_PATH without sanitization. A caller supplying a version string with embedded newlines could inject arbitrary entries into $GITHUB_PATH, enabling PATH hijacking.

Locations:

- `action.yml:211`
- `action.yml:213`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 27, 32, 37, 42): Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the top of the run script. Replaced all four uses of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) with the sanitized `$SAFE_SBT_RUNNER_VERSION`.

2. 'Setup PATH' step (lines 211, 213): Added `SAFE_PWD=$(printf '%s' "$PWD" | tr -d '\n\r')` after `cd "$SBT_TOOLPATH"`, then replaced `$PWD` with `$SAFE_PWD` in both the Windows and non-Windows GITHUB_PATH writes. This prevents newline injection via the caller-controlled sbt-runner-version input from propagating through sbt_toolpath into GITHUB_PATH.

