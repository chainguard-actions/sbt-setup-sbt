<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.1.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.1.24** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step maps `inputs.sbt-runner-version` into the env var `SBT_RUNNER_VERSION` and then writes it unsanitized to `$GITHUB_OUTPUT` multiple times (e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a newline-containing version string could inject arbitrary key=value pairs into the step output context.

Locations:

- `action.yml:23`
- `action.yml:27`
- `action.yml:31`
- `action.yml:36`
- `action.yml:37`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$PWD/sbt/bin` to `$GITHUB_PATH` (e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). `$PWD` is derived from `cd "$SBT_TOOLPATH"`, where `$SBT_TOOLPATH` is sourced from `steps.cache-paths.outputs.sbt_toolpath` — an output that was itself constructed from the unsanitized `inputs.sbt-runner-version` value in the previous step. No sanitization (`printf '%s' | tr -d '\n\r'`) is applied before the write, so a newline-containing input could inject additional entries into `$GITHUB_PATH`.

Locations:

- `action.yml:207`
- `action.yml:209`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (lines 23-37): Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block and replaced all uses of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes with the sanitized `$SAFE_SBT_RUNNER_VERSION`. This prevents a newline-containing version string from injecting additional key=value pairs into the step output context.

2. 'Setup PATH' step (lines 207-209): Replaced direct `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` (and Windows variant) with sanitized versions using `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` before writing to `$GITHUB_PATH`. This prevents a newline-containing path (derived from the unsanitized version input) from injecting additional entries into `$GITHUB_PATH`.

