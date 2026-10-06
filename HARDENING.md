<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (an attacker-controlled input) and then written directly to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically, the value is embedded unsanitized in the `sbt_toolpath`, `sbt_cachekey`, and `sbt_diskcachekey` output lines. A malicious caller could inject newlines into the input to poison subsequent `$GITHUB_OUTPUT` entries or `$GITHUB_ENV`/`$GITHUB_PATH` reads downstream.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:36`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added a sanitization line at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. Replaced all uses of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` writes (sbt_toolpath on Windows/macOS/Linux, and sbt_cachekey) with the sanitized `$SAFE_SBT_RUNNER_VERSION` variable. This prevents newline injection attacks via the attacker-controlled `sbt-runner-version` input.

