<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes the user-controlled input `inputs.sbt-runner-version` (via the env var `$SBT_RUNNER_VERSION`) directly into `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker supplying a version string containing newlines could inject arbitrary key=value pairs into the GitHub Actions output context. Affected writes include `sbt_toolpath`, `sbt_cachekey`, and `sbt_diskcachekey` outputs.

Locations:

- `action.yml:22`
- `action.yml:28`
- `action.yml:34`
- `action.yml:40`
- `action.yml:44`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$PWD/sbt/bin` to `$GITHUB_PATH` after `cd "$SBT_TOOLPATH"`, where `$SBT_TOOLPATH` is sourced from `steps.cache-paths.outputs.sbt_toolpath` — a value derived from the user-controlled `inputs.sbt-runner-version`. No sanitization (`printf '%s' | tr -d '\n\r'`) is applied before the write, allowing a crafted version string containing newlines to inject arbitrary entries into `$GITHUB_PATH`.

Locations:

- `action.yml:196`
- `action.yml:198`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION at the start of the run block with `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and replaced all GITHUB_OUTPUT writes that used $SBT_RUNNER_VERSION to use $SAFE_SBT_RUNNER_VERSION instead. This covers sbt_toolpath, sbt_cachekey, and sbt_diskcachekey outputs.
2. 'Setup PATH' step: Added sanitization of the path value before writing to $GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` for both Windows and non-Windows branches.

