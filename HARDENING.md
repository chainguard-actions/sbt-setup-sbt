<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes the value of $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via the env: block) to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. Affected lines include: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows, macOS, and Linux branches) and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. Additionally, the 'Setup PATH' step writes `$PWD/sbt/bin` to $GITHUB_PATH after `cd "$SBT_TOOLPATH"`, where $SBT_TOOLPATH is derived from the same unsanitized inputs.sbt-runner-version value propagated through steps.cache-paths.outputs.sbt_toolpath — also without sanitization.

Locations:

- `action.yml:22`
- `action.yml:26`
- `action.yml:30`
- `action.yml:35`
- `action.yml:39`
- `action.yml:43`
- `action.yml:47`
- `action.yml:50`
- `action.yml:51`
- `action.yml:196`
- `action.yml:198`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml at two locations:
1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run script, then replaced all uses of $SBT_RUNNER_VERSION in GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION. This covers all 8 affected echo statements (Windows, macOS, Linux toolpath lines, and the sbt_cachekey line).
2. 'Setup PATH' step: Added sanitization of the path value before writing to $GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` for both Windows and non-Windows branches, preventing newline injection via the SBT_TOOLPATH-derived path.

