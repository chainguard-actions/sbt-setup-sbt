<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.4** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Set up cache paths' step sets SBT_RUNNER_VERSION from inputs.sbt-runner-version via env:, then writes it unsanitized to $GITHUB_OUTPUT multiple times without the required sanitization step (printf '%s' ... | tr -d '\n\r'). For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE/sbt/$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-$RUNNER_ARCH-sbt-runner-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. An attacker-controlled input containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps.

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/action.yml. Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` as the first line of the 'Set up cache paths' run script. This sanitizes the attacker-controlled `inputs.sbt-runner-version` value (passed via env: as SBT_RUNNER_VERSION) by stripping all newline and carriage return characters before the variable is used in multiple `echo ... >> "$GITHUB_OUTPUT"` statements. Without this fix, an attacker could inject newlines into the version string to poison subsequent steps via GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. script-injection (line 89): Added `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` at the start of the 'Download and Install sbt' step's run block to sanitize the version string before it's interpolated into curl URLs, matching the sanitization pattern already used in the first step.
2. github-env-injection (line 248): In the 'Setup PATH' step, wrapped both the Windows and Linux path values in `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_PATH, preventing newline injection from a malicious sbt-runner-version input that could propagate through the sbt_toolpath output into $PWD.

