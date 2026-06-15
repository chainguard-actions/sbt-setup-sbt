<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version (user-controlled) via the env: block, then writes it unsanitized to $GITHUB_OUTPUT multiple times. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. An attacker supplying a newline-containing value for sbt-runner-version could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps.

Locations:

- `action.yml:23`
- `action.yml:27`
- `action.yml:31`
- `action.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set up cache paths' step in action.yml by sanitizing the user-controlled SBT_RUNNER_VERSION input before writing to $GITHUB_OUTPUT. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, and replaced all four uses of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes (sbt_toolpath on Windows, sbt_toolpath on macOS/Linux, sbt_cachekey) with `$SAFE_SBT_RUNNER_VERSION`. This prevents newline injection attacks that could poison subsequent steps via GITHUB_OUTPUT.

