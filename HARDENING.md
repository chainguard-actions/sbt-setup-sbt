<!-- markdownlint-disable -->

# Hardening Report: sbt--setup-sbt/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **sbt--setup-sbt/v1.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Set up cache paths' step, the env var SBT_RUNNER_VERSION is set from the untrusted input `${{ inputs.sbt-runner-version }}` and then written unsanitized to $GITHUB_OUTPUT multiple times. For example: `echo "sbt_toolpath=$RUNNER_TOOL_CACHE\\sbt\\$SBT_RUNNER_VERSION" >> "$GITHUB_OUTPUT"` and `echo "sbt_cachekey=$RUNNER_OS-sbt-$SBT_RUNNER_VERSION-$SBT_CACHE_KEY_VERSION" >> "$GITHUB_OUTPUT"`. A caller-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r'`) is absent before every write.

Locations:

- `action.yml:27`
- `action.yml:32`
- `action.yml:37`
- `action.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Added sanitization of SBT_RUNNER_VERSION at the start of the 'Set up cache paths' run script. The line `SBT_RUNNER_VERSION=$(printf '%s' "$SBT_RUNNER_VERSION" | tr -d '\n\r')` strips all newlines and carriage returns from the caller-controlled input before it is written to $GITHUB_OUTPUT in the sbt_toolpath and sbt_cachekey echo statements. This prevents newline injection attacks that could inject arbitrary key=value pairs into GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In the 'Download and Install sbt' step, added sanitization of SBT_RUNNER_VERSION at the start of the run script: (1) strip newlines/carriage-returns with 'tr -d \n\r', and (2) validate against a strict allowlist regex '^[A-Za-z0-9._-]+$' that rejects shell metacharacters ($, (, ), backticks, etc.) before the value is interpolated into curl URLs and the unzip command. This mirrors the sanitization already present in the 'Set up cache paths' step and prevents command injection via a malicious sbt-runner-version input.

