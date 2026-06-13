<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is sourced from `inputs.sbt-runner-version` (user-controlled) via the `env:` block, and then written to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A malicious caller could supply a value containing newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. This is a case (d) violation: indirect write of inputs via env var without sanitization.

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added a sanitization line at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All occurrences where `$SBT_RUNNER_VERSION` was written to `$GITHUB_OUTPUT` (in the sbt_toolpath and sbt_cachekey outputs) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead, preventing a malicious caller from injecting arbitrary key=value pairs into GITHUB_OUTPUT via embedded newlines.

