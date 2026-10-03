<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step writes the env var $SBT_RUNNER_VERSION (sourced from the untrusted input `inputs.sbt-runner-version`) to $GITHUB_OUTPUT without sanitization. Multiple echo statements like `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"` write attacker-controlled data directly to the special environment file. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent. A caller supplying a version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:14`

### github-env-injection (severity: high)

The 'Setup PATH' step writes `$PWD/sbt/bin` (derived from `$SBT_TOOLPATH`, which is sourced from `steps.cache-paths.outputs.sbt_toolpath` — itself built from the untrusted `inputs.sbt-runner-version`) to $GITHUB_PATH without sanitization. The statements `echo "$PWD\\sbt\\bin" >> "$GITHUB_PATH"` and `echo "$PWD/sbt/bin" >> "$GITHUB_PATH"` propagate attacker-influenced path data into the special environment file. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:

1. 'Set up cache paths' step (line 14): Added sanitization of SBT_RUNNER_VERSION at the start of the run script: `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`. This strips newlines before the variable is used in multiple `echo ... >> "$GITHUB_OUTPUT"` statements.

2. 'Setup PATH' step (line 196): Replaced direct writes to $GITHUB_PATH with sanitized versions using `safe_path=$(printf '%s' "$PWD/..." | tr -d '\n\r')` for both Windows and non-Windows branches, preventing newline injection via the attacker-controlled sbt-runner-version input that flows through sbt_toolpath into the path written to GITHUB_PATH.

