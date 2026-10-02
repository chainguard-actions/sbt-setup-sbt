<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). An attacker-controlled value for `sbt-runner-version` containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`. The required sanitization (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every write.

Locations:

- `action.yml:19`
- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:41`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, `steps.cache-paths.outputs.sbt_toolpath` (a `steps.*.outputs.*` value, treated as untrusted) is mapped to the env var `SBT_TOOLPATH`. The run block executes `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` (derived from the untrusted `SBT_TOOLPATH`) to `$GITHUB_PATH` without sanitization (e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). A newline-containing path could inject arbitrary entries into `$GITHUB_PATH`. The required sanitization step is absent.

Locations:

- `action.yml:11384`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block to sanitize the user-controlled input before it is embedded in multiple $GITHUB_OUTPUT writes (sbt_toolpath, sbt_cachekey, etc.).
2. 'Setup PATH' step: Added `SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` to sanitize the step output value before use, and wrapped the derived path in a `safe_path=$(printf '%s' "..." | tr -d '\n\r')` sanitization before writing to $GITHUB_PATH.

