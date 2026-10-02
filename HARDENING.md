<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes the user-controlled input `inputs.sbt-runner-version` (via env var `$SBT_RUNNER_VERSION`) to `$GITHUB_OUTPUT` multiple times without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A caller can inject newlines into the version string to poison subsequent output variables.

Locations:

- `action.yml:26`

### github-env-injection (severity: high)

The 'Setup PATH' step writes a path derived from `steps.cache-paths.outputs.sbt_toolpath` (which contains the user-controlled `inputs.sbt-runner-version`) to `$GITHUB_PATH` without sanitization. The step does `cd "$SBT_TOOLPATH"` then `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"`. Because `$SBT_TOOLPATH` is attacker-influenced, `$PWD` after the `cd` is also attacker-influenced, and the unsanitized value is written to `$GITHUB_PATH`, allowing PATH injection.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step: Added `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` to sanitize the user-controlled `inputs.sbt-runner-version` before using it in all $GITHUB_OUTPUT writes (sbt_toolpath for all OS branches and sbt_cachekey).

2. 'Setup PATH' step: Added `SAFE_PWD=$(printf '%s' "$PWD" | tr -d '\n\r')` to sanitize the $PWD value (which is derived from the user-controlled sbt_toolpath) before writing it to $GITHUB_PATH. Both Windows and non-Windows branches now use the sanitized $SAFE_PWD variable.

