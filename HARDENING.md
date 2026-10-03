<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the env var SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version, a caller-controlled input) is written to $GITHUB_OUTPUT multiple times without the required newline-stripping sanitization (printf '%s' ... | tr -d '\n\r'). An attacker-supplied version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Affected lines include the sbt_toolpath writes (Windows, macOS, Linux branches) and the sbt_cachekey write. Example failing pattern: echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:26`
- `action.yml:30`
- `action.yml:35`
- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in the 'Set up cache paths' step of action.yml. Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block to strip embedded newlines/carriage returns from the caller-controlled input. Replaced all four uses of $SBT_RUNNER_VERSION in $GITHUB_OUTPUT writes (Windows sbt_toolpath, macOS sbt_toolpath, Linux sbt_toolpath, and sbt_cachekey) with the sanitized $SAFE_SBT_RUNNER_VERSION variable.

