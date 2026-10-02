<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is set from the user-controlled input `${{ inputs.sbt-runner-version }}` and then written to $GITHUB_OUTPUT multiple times without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input can inject newline characters to poison subsequent GITHUB_OUTPUT key-value pairs. Affected lines write values like `sbt_toolpath=.../$SBT_RUNNER_VERSION`, `sbt_cachekey=...$SBT_RUNNER_VERSION...`, and `sbt_diskcachekey=...$SBT_RUNNER_VERSION...` directly to `$GITHUB_OUTPUT`. Fix: sanitize the value before each write, e.g. `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and use `$safe` in the echo statements.

Locations:

- `action.yml:20`
- `action.yml:28`
- `action.yml:32`
- `action.yml:36`
- `action.yml:39`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the user-controlled `sbt-runner-version` input. Replaced all uses of `$SBT_RUNNER_VERSION` in `$GITHUB_OUTPUT` writes (lines for sbt_toolpath, sbt_cachekey) with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in hardened/action/action.yml. The SBT_TOOLPATH value (from steps.cache-paths.outputs.sbt_toolpath) is now sanitized with `printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r'` before being used to write to $GITHUB_PATH. Additionally, the fix writes the sanitized path directly to $GITHUB_PATH instead of relying on $PWD after a cd, which is more robust and avoids any potential symlink-related issues.

