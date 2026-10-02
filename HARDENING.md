<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the untrusted input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION`, then writes it directly to `$GITHUB_OUTPUT` in multiple echo statements without the required sanitization (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker-controlled version string containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. Affected writes include: `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows, macOS, and Linux branches) and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`. The fix is to sanitize the value before each write: `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and use `$safe` in the echo.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization at the start of the run block: `safe_sbt_runner_version=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. Replaced all 4 uses of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT echo statements (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) with the sanitized `$safe_sbt_runner_version` variable to prevent newline injection attacks.

