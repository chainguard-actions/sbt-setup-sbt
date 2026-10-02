<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `${{ inputs.sbt-runner-version }}` (a caller-controlled input) and then written to `$GITHUB_OUTPUT` multiple times without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A malicious caller could supply a value containing newlines to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the run script to sanitize the caller-controlled `sbt-runner-version` input before it is written to $GITHUB_OUTPUT multiple times (in sbt_toolpath and sbt_cachekey outputs). This prevents newline injection attacks that could overwrite subsequent step outputs.

