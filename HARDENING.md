<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var $SBT_RUNNER_VERSION is sourced from inputs.sbt-runner-version (a caller-controlled input) and written to $GITHUB_OUTPUT in multiple echo statements without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline character embedded in the input value could inject additional key=value pairs into $GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Affected statements include: echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" (Windows branch, line 27), echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" (macOS branch, line 32), echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT" (Linux branch, line 37), and echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT" (line 42).

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml 'Set up cache paths' step. Added sanitization at the top of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. Replaced all four uses of `$SBT_RUNNER_VERSION` in echo-to-GITHUB_OUTPUT statements (lines 27, 32, 37, 42) with the sanitized `$SAFE_SBT_RUNNER_VERSION` variable to prevent newline injection attacks.

