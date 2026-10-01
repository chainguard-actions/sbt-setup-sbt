<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var $SBT_RUNNER_VERSION (sourced from inputs.sbt-runner-version, a caller-controlled value) is written directly into $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' ... | tr -d '\n\r'). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`. A version string containing embedded newlines could inject arbitrary key=value pairs into the step output context, potentially poisoning downstream steps.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:36`
- `action.yml:41`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var $SBT_TOOLPATH (sourced from steps.cache-paths.outputs.sbt_toolpath, which itself embeds the caller-controlled inputs.sbt-runner-version) is written directly into $GITHUB_PATH without the required sanitization step (printf '%s' ... | tr -d '\n\r'). For example: `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`. Because $SBT_TOOLPATH is set from the unsanitized output of the prior step, a version string containing newlines could inject arbitrary entries into the runner's PATH.

Locations:

- `action.yml:196`
- `action.yml:198`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step (lines 27-42): Added sanitization of SBT_RUNNER_VERSION via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before using it in all GITHUB_OUTPUT writes (sbt_toolpath and sbt_cachekey lines).
2. 'Setup PATH' step (lines 196-198): Added sanitization of SBT_TOOLPATH via `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and used the sanitized value for the `cd` command, so that `$PWD` (and thus the GITHUB_PATH entry) is derived from the sanitized path.

