<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is sourced from `inputs.sbt-runner-version` (user-controlled) and written directly to `$GITHUB_OUTPUT` multiple times without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into the version input to poison subsequent step outputs. Affected writes include:
- `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows branch, ~line 28)
- `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (macOS and Linux branches, ~lines 33, 38)
- `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"` (~line 43)
The fix is to sanitize the value before use: `safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` and then use `$safe` in the echo statements.

Locations:

- `action.yml:28`
- `action.yml:33`
- `action.yml:38`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block, then replaced all four uses of `$SBT_RUNNER_VERSION` in echo-to-GITHUB_OUTPUT statements (Windows toolpath ~line 28, macOS toolpath ~line 33, Linux toolpath ~line 38, and cachekey ~line 43) with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection attacks.

