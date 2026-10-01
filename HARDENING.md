<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets `SBT_RUNNER_VERSION` from `inputs.sbt-runner-version` (an attacker-controlled value) in its env block, then writes it unsanitized to `$GITHUB_OUTPUT` in multiple echo statements — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. None of these writes are preceded by the required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). A caller supplying a version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps that consume those outputs.

Locations:

- `action.yml:15`
- `action.yml:27`
- `action.yml:32`
- `action.yml:39`
- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of the attacker-controlled `SBT_RUNNER_VERSION` input at the start of the 'Set up cache paths' step's run block. A new variable `SAFE_SBT_RUNNER_VERSION` is created using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` to strip any embedded newlines or carriage returns. All four locations that write the version to `$GITHUB_OUTPUT` (sbt_toolpath on Windows/macOS/Linux, and sbt_cachekey) now use `$SAFE_SBT_RUNNER_VERSION` instead of the raw `$SBT_RUNNER_VERSION`, preventing GITHUB_OUTPUT injection via newline-containing version strings.

