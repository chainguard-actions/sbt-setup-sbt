<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is populated from the untrusted input `${{ inputs.sbt-runner-version }}` and then written directly into $GITHUB_OUTPUT multiple times (e.g., `echo "sbt_toolpath=.../$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=...-$SBT_RUNNER_VERSION-..." >> "$GITHUB_OUTPUT"`). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of these writes. A caller-controlled newline in the version string could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `action.yml:22`

### github-env-injection (severity: high)

In the 'Setup PATH' step, the env var SBT_TOOLPATH is populated from `${{ steps.cache-paths.outputs.sbt_toolpath }}` (a steps.*.outputs.* value that is itself derived from the untrusted input `inputs.sbt-runner-version`). The run script does `cd "$SBT_TOOLPATH"` and then writes `$PWD/sbt/bin` to $GITHUB_PATH without any `tr -d '\n\r'` sanitization. A newline embedded in the path could inject an arbitrary additional entry into the runner's PATH.

Locations:

- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in action.yml:
1. 'Set up cache paths' step: Added sanitization of SBT_RUNNER_VERSION via `SAFE_SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` before all GITHUB_OUTPUT writes that include the version string (sbt_toolpath and sbt_cachekey outputs).
2. 'Setup PATH' step: Added sanitization of the path before writing to GITHUB_PATH via `safe_path=$(printf '%s' "$PWD/sbt/bin" | tr -d '\n\r')` (both Windows and non-Windows branches), preventing newline injection into the runner's PATH.

