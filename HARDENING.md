<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled version string containing newline characters could inject arbitrary key=value pairs into the GitHub Actions output context. Affected lines include writes such as: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`
- `action.yml:43`
- `action.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Set up cache paths' step of action.yml, added a sanitization line at the start of the run script: `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All six echo statements that write to $GITHUB_OUTPUT and previously used $SBT_RUNNER_VERSION now use $SAFE_SBT_RUNNER_VERSION instead, preventing newline injection via attacker-controlled version strings.

