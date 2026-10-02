<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` multiple times — e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller-controlled value containing newlines could inject arbitrary key=value pairs into the GitHub Actions output context, potentially overwriting outputs consumed by later steps.

Locations:

- `action.yml:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added sanitization of the caller-controlled `SBT_RUNNER_VERSION` value at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All writes to $GITHUB_OUTPUT that previously used `$SBT_RUNNER_VERSION` (sbt_toolpath and sbt_cachekey outputs) now use `$SAFE_SBT_RUNNER_VERSION` instead, preventing newline injection attacks that could overwrite arbitrary output context values.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml (around line 246). The SBT_TOOLPATH value from steps.cache-paths.outputs.sbt_toolpath is now sanitized with `printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r'` before being used in `cd`. Additionally, the computed path written to $GITHUB_PATH (both Windows and non-Windows variants) is also sanitized with `printf '%s' ... | tr -d '\n\r'` before the `echo ... >> "$GITHUB_PATH"` write, preventing newline injection that could poison $GITHUB_PATH with additional entries.

