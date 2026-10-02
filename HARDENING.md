<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.6** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step of action.yml, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` in multiple places: as part of `sbt_toolpath` (lines 27, 32, 37) and `sbt_cachekey` (line 42). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller supplying a newline-containing version string could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially poisoning downstream step outputs.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the 'Set up cache paths' step's run script. This sanitizes the user-controlled `inputs.sbt-runner-version` value (mapped to the `SBT_RUNNER_VERSION` env var) by stripping all newline and carriage return characters before the value is used in any writes to $GITHUB_OUTPUT (sbt_toolpath on lines 27/32/37 and sbt_cachekey on line 42). This prevents newline injection attacks that could poison downstream step outputs.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Setup PATH' step in action.yml to sanitize path values before writing to $GITHUB_PATH. Both the Windows path ($PWD\\sbt\\bin) and Unix path ($PWD/sbt/bin) are now sanitized using `safe=$(printf '%s' "$PATH_VALUE" | tr -d '\n\r')` before being echoed to $GITHUB_PATH. This prevents newline injection via the user-controlled sbt-runner-version input.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the 'Download and Install sbt' step's run block. This sanitizes the attacker-controlled input by stripping newline/carriage-return characters before the value is used in curl URL construction and unzip commands. The fix mirrors the identical sanitization pattern already present in the 'Set up cache paths' step.

