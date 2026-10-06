<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is sourced from `${{ inputs.sbt-runner-version }}` (a user-controlled input) and then written unsanitized to `$GITHUB_OUTPUT` multiple times. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. An attacker-controlled value containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially poisoning subsequent steps. The required sanitization (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is not applied before any of these writes.

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the user-controlled SBT_RUNNER_VERSION input before writing to $GITHUB_OUTPUT. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, and replaced all occurrences of $SBT_RUNNER_VERSION in GITHUB_OUTPUT writes (sbt_toolpath across all OS branches, and sbt_cachekey) with $SAFE_SBT_RUNNER_VERSION. This prevents an attacker from injecting arbitrary key=value pairs into $GITHUB_OUTPUT via newlines embedded in the sbt-runner-version input.

