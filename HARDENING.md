<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it directly into `$GITHUB_OUTPUT` multiple times (e.g. `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`) without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). Because `inputs.sbt-runner-version` is caller-controlled, a malicious caller could inject newlines into the value to smuggle arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs and influencing downstream steps.

Locations:

- `action.yml:22`
- `action.yml:25`
- `action.yml:29`
- `action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Set up cache paths' step of action.yml. Added a sanitization line at the top of the run block: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four occurrences where `$SBT_RUNNER_VERSION` was written to `$GITHUB_OUTPUT` (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, and sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead, preventing newline injection attacks from caller-controlled input.

