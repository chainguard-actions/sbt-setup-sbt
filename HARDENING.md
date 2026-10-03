<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var `SBT_RUNNER_VERSION` (sourced from `inputs.sbt-runner-version`) is written unsanitized into `$GITHUB_OUTPUT` multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. An attacker-controlled `sbt-runner-version` input containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. Additionally, in the 'Setup PATH' step, `$PWD/sbt/bin` (where `$PWD` is derived from `cd "$SBT_TOOLPATH"` and `$SBT_TOOLPATH` originates from the unsanitized `steps.cache-paths.outputs.sbt_toolpath`) is written to `$GITHUB_PATH` without sanitization.

Locations:

- `action.yml:19`
- `action.yml:26`
- `action.yml:38`
- `action.yml:50`
- `action.yml:54`
- `action.yml:55`
- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml:
1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the run block. Replaced all occurrences of `$SBT_RUNNER_VERSION` in GITHUB_OUTPUT writes with `$SAFE_SBT_RUNNER_VERSION` to prevent newline injection from the attacker-controlled `sbt-runner-version` input.
2. 'Setup PATH' step: Added `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and used it for the `cd` command. Sanitized the final path string before writing to `$GITHUB_PATH` using `safe_path=$(printf '%s' "..." | tr -d '\n\r')` in both Windows and non-Windows branches.

