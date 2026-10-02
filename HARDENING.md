<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (a caller-controlled input) and written to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Affected writes include: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows branch, line 27), `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (macOS branch, line 32), `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Linux branch, line 37), and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"` (line 42). The fix is to sanitize the value before each write: `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and use `$safe` in the echo commands.

Locations:

- `action.yml:19`
- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' run block: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. Replaced all four uses of $SBT_RUNNER_VERSION in GITHUB_OUTPUT writes (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) with $SAFE_SBT_RUNNER_VERSION to prevent newline injection attacks from attacker-controlled version strings.

