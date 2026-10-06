<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps the attacker-controlled input `inputs.sbt-runner-version` to the env var `SBT_RUNNER_VERSION`, then writes it directly into `$GITHUB_OUTPUT` multiple times without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). A newline character embedded in the version string could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Affected writes:
- Line 27: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows branch)
- Line 32: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (macOS branch)
- Line 38: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Linux branch)
- Line 43: `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:38`
- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added a sanitization line `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all 4 occurrences of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes (lines 27, 32, 38, and 43) with the sanitized `$SAFE_SBT_RUNNER_VERSION` variable. This prevents newline injection attacks via the attacker-controlled `inputs.sbt-runner-version` input.

