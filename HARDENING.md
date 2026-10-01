<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `$SBT_RUNNER_VERSION` (sourced from `inputs.sbt-runner-version`, an attacker-controllable input) is written to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A newline character embedded in `inputs.sbt-runner-version` could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by downstream steps. Additionally, the 'Setup PATH' step writes a path derived from `$SBT_TOOLPATH` (itself built from the untrusted version input) to `$GITHUB_PATH` without sanitization.

Locations:

- `action.yml:28`
- `action.yml:35`
- `action.yml:41`
- `action.yml:46`
- `action.yml:48`
- `action.yml:49`
- `action.yml:99`
- `action.yml:101`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All GITHUB_OUTPUT writes that used $SBT_RUNNER_VERSION now use $SAFE_SBT_RUNNER_VERSION instead, preventing newline injection.
2. 'Setup PATH' step: Added sanitization of SBT_TOOLPATH using `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and used it for the `cd` command. Since $PWD is used for the GITHUB_PATH write (after cd), the sanitized path is used throughout, preventing newline injection into GITHUB_PATH.

