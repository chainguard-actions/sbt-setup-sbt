<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the user-controlled input `inputs.sbt-runner-version` and then written directly into $GITHUB_OUTPUT without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Affected lines include the echo statements writing sbt_toolpath (lines 27, 32, 37) and sbt_cachekey (line 42) which all embed $SBT_RUNNER_VERSION unsanitized into $GITHUB_OUTPUT.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step, added sanitization of the user-controlled SBT_RUNNER_VERSION input before writing it to $GITHUB_OUTPUT. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all four uses of $SBT_RUNNER_VERSION in echo-to-GITHUB_OUTPUT statements (sbt_toolpath for Windows/macOS/Linux at lines 27/32/37, and sbt_cachekey at line 42) with $SAFE_SBT_RUNNER_VERSION. This prevents an attacker-controlled version string containing newlines from injecting arbitrary key=value pairs into GITHUB_OUTPUT.

