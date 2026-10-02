<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is assigned to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`). Because `inputs.sbt-runner-version` is caller-controlled, a value containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning the outputs consumed by downstream steps. The required sanitization step (`safe=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:43`
- `action.yml:48`
- `action.yml:53`
- `action.yml:57`
- `action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block to sanitize the caller-controlled `inputs.sbt-runner-version` value. All seven occurrences where `$SBT_RUNNER_VERSION` was written to `$GITHUB_OUTPUT` (lines 27, 32, 37, 43, 48, 53, 57, 58) now use `$SAFE_SBT_RUNNER_VERSION` instead, preventing newline injection attacks.

