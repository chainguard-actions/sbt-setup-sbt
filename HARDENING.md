<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.4.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the user-controlled input `inputs.sbt-runner-version` (via `SBT_RUNNER_VERSION: ${{ inputs.sbt-runner-version }}`) and then written unsanitized to $GITHUB_OUTPUT multiple times — for example:
  echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"
  echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"
No sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is applied before any of these writes. A caller supplying a version string containing newlines could inject arbitrary key=value pairs into $GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Set up cache paths' step of action.yml. Added sanitization of the user-controlled SBT_RUNNER_VERSION input at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. All four occurrences where $SBT_RUNNER_VERSION was written to $GITHUB_OUTPUT (sbt_toolpath on Windows, sbt_toolpath on macOS, sbt_toolpath on Linux, and sbt_cachekey) now use the sanitized $SAFE_SBT_RUNNER_VERSION variable instead, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml: added `safe=$(printf '%s' "$SBT_TOOLPATH" | tr -d '\n\r')` and replaced `cd "$SBT_TOOLPATH"` with `cd "$safe"`. This sanitizes the untrusted `steps.cache-paths.outputs.sbt_toolpath` value by stripping newline and carriage-return characters before it is used in `cd`, preventing newline injection into `$GITHUB_PATH` via the resulting `$PWD` value.

