<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the user-controlled input `inputs.sbt-runner-version` is mapped to the env var `$SBT_RUNNER_VERSION` and then written to `$GITHUB_OUTPUT` multiple times without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A malicious caller could inject newlines into the version string to poison subsequent steps' outputs or environment.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:35`
- `action.yml:39`

### github-env-injection (severity: high)

In the 'Setup PATH' step, `steps.cache-paths.outputs.sbt_toolpath` (which embeds the user-controlled `inputs.sbt-runner-version`) is mapped to the env var `$SBT_TOOLPATH` and used in `cd "$SBT_TOOLPATH"`, after which `$PWD/sbt/bin` is written to `$GITHUB_PATH` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). A malicious caller could inject newlines into the version string to poison `$GITHUB_PATH` and hijack the PATH for subsequent steps.

Locations:

- `action.yml:218`
- `action.yml:220`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step: Sanitized the user-controlled `inputs.sbt-runner-version` (via `$SBT_RUNNER_VERSION`) by computing `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block. All `$GITHUB_OUTPUT` writes that embed the version now use the sanitized variable, with `safe_toolpath` and `safe_cachekey` intermediate variables also stripped of newlines.

2. 'Setup PATH' step: Added `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (and Windows equivalent) before writing to `$GITHUB_PATH`, preventing newline injection via the user-controlled version embedded in the tool path.

