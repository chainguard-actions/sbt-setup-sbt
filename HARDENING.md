<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. Because `inputs.sbt-runner-version` is caller-controlled, a value containing newlines could inject arbitrary key=value pairs into the output context. The required sanitization (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is not applied before any of these writes.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script. All four writes to $GITHUB_OUTPUT that included the version string (sbt_toolpath for Windows, macOS, and Linux, plus sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead of the raw `$SBT_RUNNER_VERSION`, preventing newline injection via the caller-controlled `inputs.sbt-runner-version` input.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Setup PATH' step of hardened/action/action.yml. The path values derived from $PWD (which traces back to user-controlled input `sbt-runner-version` via SBT_TOOLPATH) are now sanitized with `printf '%s' ... | tr -d '\n\r'` before being written to $GITHUB_PATH. This prevents newline injection attacks that could add arbitrary entries to the runner's PATH. Both the Windows path (`$PWD\sbt\bin`) and the Unix path (`$PWD/sbt/bin`) are sanitized before the echo to $GITHUB_PATH.

