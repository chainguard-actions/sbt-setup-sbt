<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` is populated from the caller-controlled input `${{ inputs.sbt-runner-version }}` and then written unsanitized into `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). A malicious caller could supply a value containing newlines to inject arbitrary key=value pairs into the output context. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step, added a sanitization line at the top of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four writes to $GITHUB_OUTPUT that included the version string (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, and sbt_cachekey) now use the sanitized `$SAFE_SBT_RUNNER_VERSION` variable instead of the raw `$SBT_RUNNER_VERSION`, preventing a malicious caller from injecting arbitrary key=value pairs via embedded newlines.

