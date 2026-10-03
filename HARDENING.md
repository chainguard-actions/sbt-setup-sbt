<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is sourced from the user-controlled input `inputs.sbt-runner-version` (via `env: SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`) and then written unsanitized to $GITHUB_OUTPUT multiple times — e.g., `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A calling workflow can supply a version string containing newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps.

Locations:

- `action.yml:19`
- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is set from `${{ steps.cache-paths.outputs.sbt_toolpath }}` — a step output that was itself constructed from the unsanitized user-controlled input `inputs.sbt-runner-version`. The script then runs `cd "$SBT_TOOLPATH"` and writes `$PWD/sbt/bin` to $GITHUB_PATH without sanitization (e.g., `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`). Because $PWD is derived from the tainted $SBT_TOOLPATH, a newline-containing version string can inject arbitrary entries into GITHUB_PATH, allowing PATH hijacking in subsequent steps.

Locations:

- `action.yml:200`
- `action.yml:205`
- `action.yml:207`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Sanitized SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`, then replaced all uses of the raw variable in GITHUB_OUTPUT writes with the sanitized version.
2. 'Setup PATH' step: Sanitized SBT_TOOLPATH using `SAFE_SBT_TOOLPATH=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` before the `cd` command, and further sanitized the resulting path before writing to GITHUB_PATH using `safe_path=$(printf '%s' "$PWD/..." | tr -d '\n\r')`.

