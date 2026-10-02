<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from inputs.sbt-runner-version (a caller-controlled input) and then written unsanitized to $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A newline embedded in the input value could inject arbitrary key=value pairs into GITHUB_OUTPUT. Similarly, in the 'Setup PATH' step, $PWD (resolved from SBT_TOOLPATH, which is derived from the same input-controlled sbt_toolpath output) is written to $GITHUB_PATH without sanitization.

Locations:

- `action.yml:26`
- `action.yml:31`
- `action.yml:35`
- `action.yml:40`
- `action.yml:44`
- `action.yml:45`
- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in two places in action.yml:
1. 'Set up cache paths' step: Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script to sanitize the caller-controlled input before it is embedded in any `echo ... >> "$GITHUB_OUTPUT"` writes.
2. 'Setup PATH' step: Replaced direct `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` with a sanitized form using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` followed by `echo "$safe_path" >> "$GITHUB_PATH"` (both Windows and non-Windows branches), preventing newline injection via the input-derived path.

