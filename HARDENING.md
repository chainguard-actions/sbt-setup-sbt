<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (an untrusted caller-controlled input) and then written directly into `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning outputs consumed by downstream steps. Affected writes include: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (lines 27, 32, 37) and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"` (line 42).

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the caller-controlled `inputs.sbt-runner-version` value. Replaced all 4 occurrences of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes (sbt_toolpath on Windows, macOS, and Linux branches, plus sbt_cachekey) with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection attacks.

