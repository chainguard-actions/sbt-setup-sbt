<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized into `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before these writes. A caller-controlled version string containing newlines could inject arbitrary key=value pairs into the GitHub Actions output context (case d violation).

Locations:

- `action.yml:18`

### github-env-injection (severity: high)

In the 'Setup PATH' step, `SBT_TOOLPATH` is set from `steps.cache-paths.outputs.sbt_toolpath` (which was derived from `inputs.sbt-runner-version`). The script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to `$GITHUB_PATH` without sanitization (`echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). A caller-controlled version string containing newlines could inject arbitrary entries into `$GITHUB_PATH`, potentially hijacking PATH resolution for subsequent steps (case d/e violation).

Locations:

- `action.yml:127`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION by stripping newlines/carriage returns via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before using it in all GITHUB_OUTPUT writes.
2. 'Setup PATH' step: Sanitized SBT_TOOLPATH before using it for `cd`, and sanitized the resulting path before writing to GITHUB_PATH using `safe_path=$(printf '%s' "..." | tr -d '\n\r')`. This prevents a caller-controlled version string containing newlines from injecting arbitrary key=value pairs into GITHUB_OUTPUT or arbitrary entries into GITHUB_PATH.

