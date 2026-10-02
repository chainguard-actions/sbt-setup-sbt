<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.11** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the user-controlled input `inputs.sbt-runner-version` and then written to $GITHUB_OUTPUT multiple times without the required sanitization (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`). An attacker who controls the `sbt-runner-version` input can inject newline characters to poison subsequent output variable names or values. Affected writes include: `echo "sbt_toolpath=...$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` (Windows, macOS, and Linux branches) and `echo "sbt_cachekey=...$SBT_RUNNER_VERSION..." >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:39`
- `action.yml:44`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is set from `steps.cache-paths.outputs.sbt_toolpath`, which was itself constructed from the user-controlled `inputs.sbt-runner-version`. The step does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` (or `$PWD\\sbt\\bin` on Windows) to $GITHUB_PATH without sanitization. Because $PWD is derived from the tainted SBT_TOOLPATH, a newline-containing version string can inject additional entries into $GITHUB_PATH, enabling PATH hijacking.

Locations:

- `action.yml:212`
- `action.yml:214`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION using `printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'` into SAFE_SBT_RUNNER_VERSION before all GITHUB_OUTPUT writes that include the version string (sbt_toolpath on Windows/macOS/Linux, and sbt_cachekey).
2. 'Setup PATH' step: Added sanitization of the path before writing to GITHUB_PATH using `printf '%s' "$PWD/sbt/bin" | tr -d '\n\r'` (and the Windows equivalent) stored in safe_path before the echo to $GITHUB_PATH.

