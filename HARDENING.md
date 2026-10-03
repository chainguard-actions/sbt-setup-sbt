<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from `inputs.sbt-runner-version` (an attacker-controlled input) and then written directly to `$GITHUB_OUTPUT` in multiple `echo` statements without the required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`). This allows a caller to inject newlines into `$GITHUB_OUTPUT`, potentially overwriting subsequent output variables. Affected lines include the `sbt_toolpath`, `sbt_cachekey`, and related writes that embed `$SBT_RUNNER_VERSION`. No `printf`/`tr -d` sanitization exists anywhere in the file.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`
- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the attacker-controlled `SBT_RUNNER_VERSION` input at the top of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four locations that previously wrote `$SBT_RUNNER_VERSION` directly to `$GITHUB_OUTPUT` (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, and sbt_cachekey) now use `$SAFE_SBT_RUNNER_VERSION` instead, preventing newline injection attacks.

