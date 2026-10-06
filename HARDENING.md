<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.23** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes `$SBT_RUNNER_VERSION` — sourced from `inputs.sbt-runner-version` via the `env:` block — to `$GITHUB_OUTPUT` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). A caller supplying a version string containing newline characters could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. Example failing lines: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:19`
- `action.yml:22`
- `action.yml:25`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$SBT_TOOLPATH` — sourced from `steps.cache-paths.outputs.sbt_toolpath` (which itself was constructed from the user-controlled `inputs.sbt-runner-version`) via the `env:` block — to `$GITHUB_PATH` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). A malicious version string containing newlines could inject arbitrary paths into `$GITHUB_PATH`, enabling PATH-hijacking attacks. Example failing line: `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`.

Locations:

- `action.yml:222`
- `action.yml:224`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION="$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')"` at the start of the run block. Replaced all three GITHUB_OUTPUT writes that used `$SBT_RUNNER_VERSION` (Windows sbt_toolpath, non-Windows sbt_toolpath, and sbt_cachekey) to use `$SAFE_SBT_RUNNER_VERSION` instead.

2. 'Setup PATH' step: Added `SAFE_SBT_TOOLPATH="$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')"` and replaced the GITHUB_PATH writes to use `$SAFE_SBT_TOOLPATH` directly (instead of `$PWD` after `cd "$SBT_TOOLPATH"`), covering both Windows and non-Windows paths. This prevents newline injection via the user-controlled sbt-runner-version input that flows through sbt_toolpath into GITHUB_PATH.

