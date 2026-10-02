<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version via the env: block) is written into $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A caller-supplied version string containing embedded newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Similarly, in the 'Setup PATH' step, $PWD (resolved after `cd "$SBT_TOOLPATH"` where SBT_TOOLPATH is derived from the tainted sbt_toolpath output containing SBT_RUNNER_VERSION) is written to $GITHUB_PATH without sanitization.

Locations:

- `action.yml:24`
- `action.yml:28`
- `action.yml:32`
- `action.yml:36`
- `action.yml:38`
- `action.yml:39`
- `action.yml:178`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection issues in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION (user-controlled input) at the start of the run block using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All GITHUB_OUTPUT writes that included the version string now use SAFE_SBT_RUNNER_VERSION instead.
2. 'Setup PATH' step: Added sanitization of the path before writing to GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and the Windows variant with backslashes), preventing embedded newlines in the tainted SBT_TOOLPATH from injecting arbitrary entries into GITHUB_PATH.

