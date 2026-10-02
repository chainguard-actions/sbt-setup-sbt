<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the input `inputs.sbt-runner-version` is mapped to the env var `SBT_RUNNER_VERSION` and then written unsanitized to `$GITHUB_OUTPUT` multiple times without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"`. An attacker-controlled version string containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs. Additionally, in the 'Setup PATH' step, `$PWD/sbt/bin` (where `$PWD` is derived from `cd "$SBT_TOOLPATH"` and `SBT_TOOLPATH` traces back to the untrusted `inputs.sbt-runner-version` via `steps.cache-paths.outputs.sbt_toolpath`) is written unsanitized to `$GITHUB_PATH`, allowing a newline-containing input to inject arbitrary entries into the runner's PATH.

Locations:

- `action.yml:27`
- `action.yml:47`
- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml:
1. 'Set up cache paths' step: Added SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r') at the start of the run script, then replaced all $SBT_RUNNER_VERSION references in GITHUB_OUTPUT writes with $SAFE_SBT_RUNNER_VERSION to prevent newline injection via the sbt-runner-version input.
2. 'Setup PATH' step: Added safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r') (and Windows equivalent) before writing to GITHUB_PATH, preventing newline injection via the path derived from the untrusted sbt-runner-version input.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Download and Install sbt' step in action.yml: added sanitization of SBT_RUNNER_VERSION at the start of the run script using `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')`, then replaced all uses of the raw `$SBT_RUNNER_VERSION` in curl URLs and unzip commands with `$SAFE_SBT_RUNNER_VERSION`. This is consistent with the sanitization pattern already used in the first step ('Set up cache paths').

