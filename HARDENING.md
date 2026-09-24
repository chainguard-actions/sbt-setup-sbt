<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets `SBT_RUNNER_VERSION` from `inputs.sbt-runner-version` in its `env:` block and then writes it to `$GITHUB_OUTPUT` multiple times without sanitization. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. This is a case (d) violation — an input-sourced env var is written to a special environment file without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a version string containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by downstream steps.

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the input-sourced `SBT_RUNNER_VERSION` env var at the top of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All three occurrences where `$SBT_RUNNER_VERSION` was written to `$GITHUB_OUTPUT` (sbt_toolpath in Windows/macOS/Linux branches, and sbt_cachekey) now use `$SAFE_SBT_RUNNER_VERSION` instead, preventing newline injection attacks.

